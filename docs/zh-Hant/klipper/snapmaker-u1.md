# Snapmaker U1

本文說明如何為 **Snapmaker U1** 印表機編譯並安裝 iHeater 韌體。

與一般安裝流程相比，有三處不同：

1. 韌體要從 **Snapmaker 的 Klipper 分支**（[u1-klipper](https://github.com/Snapmaker/u1-klipper)）編譯，而不是主線 Klipper。Snapmaker 修改了約 20% 的 Klipper 程式碼，因此 MCU 韌體必須與印表機上的 Klipper 版本相同。
2. 在 Snapmaker 分支中，使用 USB 的 STM32F042 **不做小幅修正就無法編譯成功**。修正方式見下文。
3. 韌體在**電腦**上編譯，而不是在印表機上，並透過 **STM32CubeProgrammer** 以 DFU 模式燒錄到 iHeater。**不使用** Katapult Bootloader。

後續設定（組態、巨集、校準）與任何使用 Klipper 的印表機相同：請參閱[組態](configuration.md)。

---

## 所需準備

- iHeater 板（STM32F042F6P6）以及**支援資料傳輸**的 USB 線
- Snapmaker U1，並可透過 SSH 連線（帳號 `root`，密碼 `snapmaker`）
- 用於編譯韌體的電腦：
    - **Windows** — WSL（在 Windows 中執行的 Ubuntu），本文主要以此為例
    - **Linux** — 指令相同，可略過 WSL 相關段落
    - **macOS** — 任何裝有 Ubuntu 的虛擬機器（UTM、Parallels、VirtualBox），或直接在 macOS 上編譯，依個人習慣即可。做法相同，Ubuntu 內的指令也完全一樣。
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)（免費，支援 Windows / macOS / Linux）

---

## 1. 查詢印表機的韌體版本

在印表機螢幕或 Fluidd（系統資訊區塊）中找到韌體版本，例如 `1.4.1`。稍後選擇 Klipper 版本時會用到。

---

## 2. 準備 Ubuntu

#### Windows (WSL)

以**系統管理員身分開啟 PowerShell** 並執行：

```powershell
wsl --install -d Ubuntu-24.04
```

若 Windows 要求重新開機，請重新啟動電腦。首次啟動 Ubuntu 時，請設定使用者名稱和密碼。

之後所有指令都在 **Ubuntu 視窗中**執行。

#### macOS / Linux

啟動裝有 Ubuntu 的虛擬機器（Linux 直接開啟終端機即可）。之後所有指令都在其中執行。

安裝編譯工具：

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. 複製 U1 專用的 Klipper

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### 選擇版本

查看儲存庫中有哪些韌體版本：

```bash
git log --oneline | grep "Release firmware"
```

範例：

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

選擇與**印表機韌體版本相符**的 commit。若沒有完全相同的版本（例如印表機是 `1.4.1`，而清單中只有 `1.4.0`），請選擇主次版本號相同、最接近的較低版本：`1.4.1` 就選 `1.4.0`。

從所選的 commit 建立工作分支（請換成你自己的雜湊值）：

```bash
git switch -c iheater edf309af
```

!!! tip "最簡單的方式"
    將印表機更新到最新韌體，並從最新的 commit 編譯，版本就一定會一致。

---

## 4. 修正 STM32F042 的 USB

在 Snapmaker 分支中，`src/generic/usb_cdc.c` 檔案新增了一個會呼叫 `usb_clear_stall()` 的函式。而 `usb_clear_stall()` 只針對高階 STM32 的 USB 控制器（`src/stm32/usbotg.c`）實作。STM32F042 使用的是另一種 USB 控制器（`src/stm32/usbfs.c`），因此編譯時會失敗並出現以下錯誤：

```
undefined reference to `usb_clear_stall'
```

請補上缺少的函式：**完整**複製下方區塊（從 `grep` 到最後一行 `EOF`，包含該行），貼到 Ubuntu 終端機中並按 **Enter**。指令會自動將函式附加到 `src/stm32/usbfs.c` 檔案結尾，不需要手動編輯任何內容。

```bash
grep -q usb_clear_stall src/stm32/usbfs.c || cat >> src/stm32/usbfs.c << 'EOF'

/****************************************************************
 * Snapmaker usb_cdc.c CLEAR_FEATURE(ENDPOINT_HALT) support
 ****************************************************************/

#include "board/irq.h" // irq_save

// Clear STALL condition on an endpoint (port of usbotg.c version)
void
usb_clear_stall(uint32_t ep, int is_in)
{
    irqstatus_t flag = irq_save();
    uint32_t epr = USB_EPR[ep];
    if (is_in) {
        if ((epr & USB_EPTX_STAT) == USB_EP_TX_STALL)
            USB_EPR[ep] = calc_epr_bits(epr, USB_EPTX_STAT, USB_EP_TX_NAK);
    } else {
        if ((epr & USB_EPRX_STAT) == USB_EP_RX_STALL)
            USB_EPR[ep] = calc_epr_bits(epr, USB_EPRX_STAT, USB_EP_RX_VALID);
    }
    irq_restore(flag);
}
EOF
```

驗證：

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

應該會找到包含 `usb_clear_stall(uint32_t ep, int is_in)` 的那一行。

!!! note
    指令以 `grep -q ... ||` 開頭：如果 Snapmaker 在新版本中自行修正了這個問題，函式就不會被重複加入。

為了避免韌體版本帶有 `dirty` 標記，請將此變更提交為 commit：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. 設定與編譯

```bash
make menuconfig
```

選擇：

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    不使用 Katapult，韌體從快閃記憶體的起始位址（`0x08000000`）開始寫入。若保留 `8KiB bootloader`，燒錄後板子將無法開機。

在 `Optional features` 區段中關閉所有不需要的項目（STM32F042 的快閃記憶體只有 32 KB）：

```
[*] Support micro-controller based ADC (analog to digital)
[ ] Support communicating with external chips via SPI bus
[ ] Support communicating with external chips via I2C bus
[ ] Support software based I2C "bit-banging"
[ ] Support software based SPI "bit-banging"
[*] Support GPIO based button reading
[ ] Support Trinamic stepper motor driver UART communication
[ ] Support 'neopixel' type LED control
[ ] Support measuring fan tachometer GPIO pins
```

清單中若還有其他項目（感測器、顯示器、外部 ADC），也請一併關閉。

儲存（`Q` → `Y`）並編譯：

```bash
make clean
```
```bash
make
```

結果：

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

編譯只需幾秒鐘，這是正常的。

檢查大小（非必要）：

```bash
arm-none-eabi-size out/klipper.elf
```

`text` + `data` 的總和必須小於 32768。

!!! tip "儲存設定"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    下次可直接執行：`cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. 將韌體複製到電腦

#### Windows (WSL)

在 Windows 檔案總管中開啟韌體所在資料夾：

```bash
explorer.exe out
```

或將檔案複製到桌面（將 `YourUserName` 替換為你的 Windows 使用者名稱）：

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### 虛擬機器

若虛擬機器已設定共用資料夾，可直接透過共用資料夾傳輸；若沒有，則透過 SSH。在主機上執行：

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

若沒有 SSH，可以用臨時網頁伺服器。在虛擬機器中執行：

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

在瀏覽器中開啟 `http://VM_IP:8000` 並下載 `klipper.bin`。按 `Ctrl+C` 即可停止伺服器。

虛擬機器的 IP 查詢方式：`ip -4 addr | grep inet`

---

## 7. 透過 STM32CubeProgrammer 燒錄（DFU）

### 進入 DFU 模式

視板子的硬體版本而定，需透過**跳線帽**或**按鈕**進入 DFU 模式：

- **BOOT0 跳線帽** — 插上跳線帽，並以 USB 將板子連接到電腦（若板子已連接，請按 RESET）。燒錄完成後必須拔掉跳線帽。
- **BOOT 按鈕** — 按住按鈕，以 USB 將板子連接到電腦（或按 RESET），然後放開 BOOT。

### 燒錄

1. 啟動 STM32CubeProgrammer。
2. 在右側選擇 **USB** 介面，按下重新整理按鈕 — **Port** 欄位會出現 `USB1`。
3. 按下 **Connect**。
4. 在左側開啟 **Erasing & Programming** 分頁。
5. **File path** — 選擇 `klipper.bin`。
6. **Start address** — `0x08000000`。
7. 勾選 **Verify programming**。
8. 按下 **Start Programming**。應會出現 `File download complete` 訊息。
9. 按下 **Disconnect**，拔掉 BOOT0 跳線帽（若你的板子上有），然後重新連接板子。

!!! note "Windows：CubeProgrammer 中看不到板子"
    DFU 驅動程式會隨 STM32CubeProgrammer 一併安裝。若在裝置管理員中板子顯示為帶有黃色圖示的 `STM32 BOOTLOADER`，請從 CubeProgrammer 資料夾重新安裝驅動程式：`Drivers\DFU_Driver`。

!!! note "若板子上原本有 Katapult"
    Katapult 會被覆寫，這是正常的：韌體編譯時不含 Bootloader，會從快閃記憶體開頭開始存放。

---

## 8. 連接到 U1

將 iHeater 連接到印表機的 USB 埠，並透過 SSH 登入印表機（帳號 `root`，密碼 `snapmaker`）：

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    印表機重新燒錄韌體或 IP 變更後會出現此訊息。請刪除舊的主機金鑰後重新連線：
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

查詢板子的 ID：

```bash
ls /dev/serial/by-id/
```

結果：

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. 組態

之後與一般 Klipper 相同 — 請參閱[組態](configuration.md)：

1. 將 `iHeater.cfg` 放在 `printer.cfg` 旁邊（透過 Fluidd 的組態編輯器或 SSH）。
2. 在 `printer.cfg` 中加入：
    ```ini
    [include iHeater.cfg]
    ```
3. 在 `iHeater.cfg` 中指定板子的 ID：
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. 重新啟動 Klipper（`FIRMWARE_RESTART`）。

!!! warning "更新印表機韌體"
    更新 U1 韌體後，請確認 `printer.cfg` 中仍保留 `[include iHeater.cfg]`。若印表機上的 Klipper 版本已變更，請從對應的 commit 重新編譯 iHeater 韌體（步驟 3–7）。

---

## 疑難排解

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
iHeater 韌體版本與印表機上的 Klipper 不一致。請從與 U1 韌體版本對應的 commit 重新編譯（步驟 3）。

**板子沒有出現在 `/dev/serial/by-id/`**

- 韌體是以 `8KiB bootloader` 編譯的 — 請改用 `No bootloader` 重新編譯。
- 未拔掉 BOOT0 跳線帽 — 板子仍停留在 DFU 模式。
- USB 線僅能充電，不支援資料傳輸。
- 確認印表機是否偵測得到該裝置：`dmesg | tail -20`。

**編譯時出現 `undefined reference to 'usb_clear_stall'`**
未執行步驟 4。

**編譯時出現 `region FLASH overflowed`**
快閃記憶體空間不足：請關閉 `Optional features` 中多餘的項目（步驟 5）。

**`gcode command M141 already registered`**
同名巨集已在印表機的其他設定檔中定義。請將其中一個註解掉。

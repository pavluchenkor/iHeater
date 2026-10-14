# Snapmaker U1

适用于 **Snapmaker U1** 打印机的 iHeater 固件编译与安装说明。

与常规安装相比，有三处不同：

1. 固件从 **Snapmaker 的 Klipper 分支**（[u1-klipper](https://github.com/Snapmaker/u1-klipper)）编译，而不是从主线 Klipper 编译。Snapmaker 修改了约 20% 的 Klipper 代码，因此 MCU 固件必须与打印机上的 Klipper 版本一致。
2. 在 Snapmaker 的分支中，使用 USB 通信的 STM32F042 固件**不做一个小修复就无法编译通过**。修复方法见下文。
3. 固件在**电脑**上编译，而不是在打印机上编译，并通过 **STM32CubeProgrammer** 以 DFU 模式烧录到 iHeater。**不使用** Katapult 引导程序。

后续配置（配置文件、宏、校准）与任何 Klipper 打印机相同：参见[配置](configuration.md)。

---

## 所需准备

- iHeater 板（STM32F042F6P6）和一根**支持数据传输**的 USB 线（不能是纯充电线）
- Snapmaker U1 及其 SSH 访问权限（用户名 `root`，密码 `snapmaker`）
- 用于编译固件的电脑：
    - **Windows**：WSL（在 Windows 中运行的 Ubuntu），本说明以此为主要方案
    - **Linux**：命令相同，跳过 WSL 部分即可
    - **macOS**：任意 Ubuntu 虚拟机（UTM、Parallels、VirtualBox），或直接在 macOS 上编译，视个人习惯而定。思路相同，Ubuntu 中的命令完全一样。
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)（免费，支持 Windows / macOS / Linux）

---

## 1. 查看打印机固件版本

在打印机屏幕上或 Fluidd 中（系统信息页面）找到固件版本，例如 `1.4.1`。选择 Klipper 版本时需要用到它。

---

## 2. 准备 Ubuntu

#### Windows (WSL)

以**管理员身份打开 PowerShell**并执行：

```powershell
wsl --install -d Ubuntu-24.04
```

如果 Windows 提示重启，请重启电脑。首次启动 Ubuntu 时设置用户名和密码。

之后所有命令都在 **Ubuntu 窗口**中执行。

#### macOS / Linux

启动 Ubuntu 虚拟机（Linux 上直接使用终端）。之后所有命令都在其中执行。

安装编译工具：

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. 克隆 U1 版 Klipper

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### 选择版本

查看仓库中有哪些固件版本：

```bash
git log --oneline | grep "Release firmware"
```

示例：

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

选择与**打印机固件版本一致**的提交。如果没有完全相同的版本（例如打印机是 `1.4.1`，而列表中只有 `1.4.0`），请选择主、次版本号相同且最接近的较低版本：`1.4.1` 对应 `1.4.0`。

基于所选提交创建工作分支（将哈希值替换为你选定的提交）：

```bash
git switch -c iheater edf309af
```

!!! tip "最简单的方案"
    将打印机升级到最新固件，并基于最新提交编译。这样版本一定一致。

---

## 4. 修复 STM32F042 的 USB 编译问题

在 Snapmaker 的分支中，`src/generic/usb_cdc.c` 文件里新增了一个调用 `usb_clear_stall()` 的函数，而 `usb_clear_stall()` 只在高端 STM32 的 USB 控制器驱动（`src/stm32/usbotg.c`）中实现。STM32F042 使用的是另一种 USB 控制器（`src/stm32/usbfs.c`），因此编译时会报错：

```
undefined reference to `usb_clear_stall'
```

补上缺失的函数：**完整**复制下面的代码块（从 `grep` 到最后一行 `EOF`，包括该行），粘贴到 Ubuntu 终端并按 **Enter**。该命令会自动把函数追加到 `src/stm32/usbfs.c` 文件末尾，无需手动编辑。

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

检查：

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

输出中应包含 `usb_clear_stall(uint32_t ep, int is_in)` 这一行。

!!! note
    命令以 `grep -q ... ||` 开头：如果 Snapmaker 在新版本中自行修复了此问题，该函数不会被重复添加。

为避免固件版本号中出现 `dirty` 标记，请提交此修改：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. 配置与编译

```bash
make menuconfig
```

选择：

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    不使用 Katapult，固件从闪存起始地址（`0x08000000`）开始烧录。如果保留 `8KiB bootloader`，烧录后板子将无法启动。

在 `Optional features` 部分关闭所有不需要的选项（STM32F042 只有 32 KB 闪存）：

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

其余选项（传感器、显示屏、外部 ADC）如果在列表中，也请关闭。

保存（`Q` → `Y`）并编译：

```bash
make clean
```
```bash
make
```

输出结果：

```
  Linking out/klipper.elf
  Creating hex file out/klipper.bin
```

编译只需几秒钟，属于正常现象。

检查大小（可选）：

```bash
arm-none-eabi-size out/klipper.elf
```

`text` + `data` 之和应小于 32768。

!!! tip "保存配置"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    下次直接执行：`cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. 将固件复制到电脑

#### Windows (WSL)

在 Windows 资源管理器中打开固件所在文件夹：

```bash
explorer.exe out
```

或者把文件复制到桌面（将 `YourUserName` 替换为 Windows 用户名）：

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### 虚拟机

如果已配置虚拟机共享文件夹，直接通过共享文件夹复制即可；否则可通过 SSH 复制。在主机上执行：

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

如果没有 SSH，可以临时启动一个 Web 服务器。在虚拟机中执行：

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

在浏览器中打开 `http://VM_IP:8000` 并下载 `klipper.bin`。按 `Ctrl+C` 停止服务器。

查看虚拟机 IP：`ip -4 addr | grep inet`

---

## 7. 通过 STM32CubeProgrammer 烧录（DFU）

### 进入 DFU 模式

根据板子的硬件版本不同，DFU 模式通过**跳线帽**或**按键**进入：

- **BOOT0 跳线帽**：插上跳线帽，再通过 USB 将板子连接到电脑（如果板子已连接，则按一下 RESET）。烧录完成后必须取下跳线帽。
- **BOOT 按键**：按住该按键，通过 USB 将板子连接到电脑（或按一下 RESET），然后松开 BOOT。

### 烧录

1. 启动 STM32CubeProgrammer。
2. 在右侧选择 **USB** 接口，点击刷新按钮，**Port** 栏中会出现 `USB1`。
3. 点击 **Connect**。
4. 在左侧打开 **Erasing & Programming** 选项卡。
5. **File path**：选择 `klipper.bin`。
6. **Start address**：`0x08000000`。
7. 勾选 **Verify programming**。
8. 点击 **Start Programming**，应显示提示 `File download complete`。
9. 点击 **Disconnect**，取下 BOOT0 跳线帽（如果你的板子上有），然后重新连接板子。

!!! note "Windows：CubeProgrammer 中看不到板子"
    DFU 驱动随 STM32CubeProgrammer 一起安装。如果设备管理器中板子显示为带黄色感叹号的 `STM32 BOOTLOADER`，请从 CubeProgrammer 安装目录中重新安装驱动：`Drivers\DFU_Driver`。

!!! note "如果板子上之前有 Katapult"
    它会被覆盖，这是正常的：固件编译时不含引导程序，会从闪存起始地址开始占用空间。

---

## 8. 连接到 U1

将 iHeater 连接到打印机的 USB 端口，并通过 SSH 登录打印机（用户名 `root`，密码 `snapmaker`）：

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    打印机重新刷机或 IP 变更后会出现此提示。删除旧密钥后重新连接即可：
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

查找板子的 ID：

```bash
ls /dev/serial/by-id/
```

输出结果：

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. 配置

后续步骤与常规 Klipper 相同，参见[配置](configuration.md)：

1. 将 `iHeater.cfg` 放在 `printer.cfg` 旁边（通过 Fluidd 中的配置编辑器或 SSH）。
2. 在 `printer.cfg` 中添加：
    ```ini
    [include iHeater.cfg]
    ```
3. 在 `iHeater.cfg` 中填写板子的 ID：
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. 重启 Klipper（`FIRMWARE_RESTART`）。

!!! warning "打印机固件更新"
    更新 U1 固件后，请检查 `printer.cfg` 中是否仍保留 `[include iHeater.cfg]`。如果打印机上的 Klipper 版本发生了变化，请从对应的提交重新编译 iHeater 固件（步骤 3–7）。

---

## 常见问题

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
iHeater 固件版本与打印机上的 Klipper 不一致。请从与 U1 固件版本对应的提交重新编译（步骤 3）。

**板子没有出现在 `/dev/serial/by-id/` 中**

- 固件使用 `8KiB bootloader` 编译：请改用 `No bootloader` 重新编译。
- 未取下 BOOT0 跳线帽：板子仍处于 DFU 模式。
- USB 线是纯充电线，不支持数据传输。
- 检查打印机是否识别到了该设备：`dmesg | tail -20`。

**编译时出现 `undefined reference to 'usb_clear_stall'`**
未完成步骤 4。

**编译时出现 `region FLASH overflowed`**
闪存放不下：请关闭 `Optional features` 中多余的选项（步骤 5）。

**`gcode command M141 already registered`**
打印机的其他配置文件中已定义同名宏。请注释掉其中一个。

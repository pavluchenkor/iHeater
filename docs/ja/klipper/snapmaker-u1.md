# Snapmaker U1

**Snapmaker U1** プリンター向けに、iHeater ファームウェアをビルドしてインストールする手順です。

通常のインストールとは次の3点が異なります。

1. ファームウェアは、メインの Klipper ではなく **Snapmaker の Klipper フォーク**（[u1-klipper](https://github.com/Snapmaker/u1-klipper)）からビルドします。Snapmaker は Klipper のコードの約 20%を変更しているため、MCU のファームウェアはプリンターと同じバージョンのものを使用する必要があります。
2. Snapmaker のフォークでは、USB 接続の STM32F042 向けビルドは**小さな修正を加えないとビルドできません**。修正内容は後述します。
3. ファームウェアは、プリンター上ではなく**コンピューター上**でビルドし、**STM32CubeProgrammer** を使用して DFU モードで iHeater に書き込みます。Katapult ブートローダーは**使用しません**。

以降の設定（コンフィグ、マクロ、キャリブレーション）は、他の Klipper プリンターと同じです。[コンフィグ](configuration.md) を参照してください。

---

## 必要なもの

- iHeater ボード（STM32F042F6P6）と、**データ線付きの** USB ケーブル
- Snapmaker U1 と、SSH によるアクセス（ユーザー名 `root`、パスワード `snapmaker`）
- ファームウェアをビルドするコンピューター：
    - **Windows** — WSL（Windows 上の Ubuntu）。この手順の基本となる方法です
    - **Linux** — コマンドは同じです。WSL の手順は飛ばしてください
    - **macOS** — Ubuntu の仮想マシン（UTM、Parallels、VirtualBox など）、または macOS 上で直接ビルドします。お好みの方法を選んでください。考え方は同じで、Ubuntu 内のコマンドに違いはありません。
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)（無料、Windows / macOS / Linux 対応）

---

## 1. プリンターのファームウェアバージョンを確認する

プリンターの画面または Fluidd（システム情報のセクション）で、ファームウェアのバージョン（例: `1.4.1`）を確認します。このバージョンは、Klipper のバージョンを選ぶ際に必要です。

---

## 2. Ubuntu の準備

#### Windows（WSL）

**PowerShell を管理者として**起動し、次を実行します。

```powershell
wsl --install -d Ubuntu-24.04
```

Windows から求められた場合は、コンピューターを再起動します。Ubuntu の初回起動時に、ユーザー名とパスワードを設定します。

以降のコマンドはすべて **Ubuntu のウィンドウ内**で実行します。

#### macOS / Linux

Ubuntu の仮想マシンを起動します（Linux の場合はターミナルを開くだけです）。以降のコマンドはすべてその中で実行します。

ビルドツールをインストールします。

```bash
sudo apt update
```
```bash
sudo apt install -y git build-essential gcc-arm-none-eabi libnewlib-arm-none-eabi binutils-arm-none-eabi libncurses-dev python3
```

---

## 3. U1 向け Klipper のクローン

```bash
cd ~
```
```bash
git clone https://github.com/Snapmaker/u1-klipper
```
```bash
cd u1-klipper
```

### バージョンの選択

リポジトリにどのファームウェアバージョンがあるかを確認します。

```bash
git log --oneline | grep "Release firmware"
```

例：

```
10f2f697 Release firmware version 1.6.0
7eb32121 Release firmware version 1.5.2
a073ed5c Release firmware version 1.5.0
edf309af Release firmware version 1.4.0
318c47e0 Release firmware version 1.3.0
```

**プリンターのファームウェアバージョンと一致する**コミットを選びます。正確なバージョンがない場合（例: プリンターが `1.4.1` で、一覧には `1.4.0` しかない場合）は、同じ系列で最も近い下位のバージョンを選びます。`1.4.1` の場合は `1.4.0` です。

選んだコミットから作業用ブランチを作成します（ハッシュはご自身のものに置き換えてください）。

```bash
git switch -c iheater edf309af
```

!!! tip "最も簡単な方法"
    プリンターを最新のファームウェアに更新し、最新のコミットからビルドします。これでバージョンが確実に一致します。

---

## 4. STM32F042 向けの USB 修正

Snapmaker のフォークでは、`src/generic/usb_cdc.c` に `usb_clear_stall()` を呼び出す関数が追加されています。この関数は、上位モデルの STM32 の USB コントローラー（`src/stm32/usbotg.c`）にのみ実装されています。STM32F042 は別の USB コントローラー（`src/stm32/usbfs.c`）を使用しているため、ビルドは次のエラーで失敗します。

```
undefined reference to `usb_clear_stall'
```

不足している関数を追加します。下のブロックを（`grep` から最後の `EOF` の行まで）**すべて**コピーし、Ubuntu のターミナルに貼り付けて **Enter** を押してください。このコマンドが関数を `src/stm32/usbfs.c` の末尾に自動で追記するため、手作業で編集する必要はありません。

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

確認：

```bash
grep -n usb_clear_stall src/stm32/usbfs.c
```

`usb_clear_stall(uint32_t ep, int is_in)` を含む行が見つかるはずです。

!!! note
    コマンドは `grep -q ... ||` で始まっています。新しいバージョンで Snapmaker 側がこの問題を修正した場合でも、関数が二重に追加されることはありません。

ファームウェアのバージョンに `dirty` の表記が付かないように、変更をコミットして保存します。

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git commit -am "stm32/usbfs: add usb_clear_stall()"
```

---

## 5. 設定とビルド

```bash
make menuconfig
```

次のように選択します。

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture (STMicroelectronics STM32)
    Processor model (STM32F042)
    Bootloader offset (No bootloader)
    Clock Reference (Internal clock)
    Communication interface (USB (on PA9/PA10))
```

!!! warning "Bootloader offset — No bootloader"
    Katapult は使用しないため、ファームウェアはフラッシュの先頭（`0x08000000`）から書き込まれます。`8KiB bootloader` のままにすると、書き込み後にボードが起動しません。

`Optional features` セクションで、不要なものをすべて無効にします（STM32F042 のフラッシュは 32 KB しかありません）。

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

その他の項目（センサー、ディスプレイ、外付け ADC）も、一覧にあれば無効にしてください。

保存（`Q` → `Y`）してからビルドします。

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

ビルドは数秒で終わりますが、これで正常です。

サイズの確認（任意）：

```bash
arm-none-eabi-size out/klipper.elf
```

`text` + `data` の合計は 32768 未満である必要があります。

!!! tip "設定を保存する"
    ```bash
    cp .config ~/iheater-u1.config
    ```
    次回からは：`cp ~/iheater-u1.config .config && make olddefconfig && make`

---

## 6. ファームウェアをコンピューターにコピーする

#### Windows（WSL）

ファームウェアのフォルダーを Windows のエクスプローラーで開きます。

```bash
explorer.exe out
```

または、ファイルをデスクトップにコピーします（`YourUserName` は Windows のユーザー名に置き換えてください）。

```bash
cp out/klipper.bin /mnt/c/Users/YourUserName/Desktop/
```

#### 仮想マシン

仮想マシンの共有フォルダーが設定されている場合は、それを使用します。設定されていない場合は SSH を使用します。メインのコンピューターで次を実行します。

```bash
scp user@VM_IP:~/u1-klipper/out/klipper.bin ~/Desktop/
```

SSH を使わない場合は、一時的な Web サーバーを使用します。仮想マシンで次を実行します。

```bash
cd ~/u1-klipper/out && python3 -m http.server 8000
```

ブラウザーで `http://VM_IP:8000` を開き、`klipper.bin` をダウンロードします。サーバーは `Ctrl+C` で停止します。

仮想マシンの IP アドレスの確認：`ip -4 addr | grep inet`

---

## 7. STM32CubeProgrammer による書き込み（DFU）

### DFU モードへの移行

ボードのリビジョンに応じて、DFU モードは**ジャンパー**または**ボタン**で有効にします。

- **BOOT0 ジャンパー** — ジャンパーを取り付け、ボードを USB でコンピューターに接続します（ボードがすでに接続されている場合は RESET を押します）。書き込み後はジャンパーを外す必要があります。
- **BOOT ボタン** — ボタンを押したままボードを USB でコンピューターに接続し（または RESET を押し）、その後 BOOT を離します。

### 書き込み

1. STM32CubeProgrammer を起動します。
2. 右側でインターフェースとして **USB** を選択し、更新ボタンを押します。**Port** フィールドに `USB1` が表示されます。
3. **Connect** を押します。
4. 左側の **Erasing & Programming** タブを開きます。
5. **File path** で `klipper.bin` を選択します。
6. **Start address** は `0x08000000` にします。
7. **Verify programming** にチェックを入れます。
8. **Start Programming** を押します。`File download complete` というメッセージが表示されます。
9. **Disconnect** を押し、BOOT0 ジャンパーを外して（お使いのボードにある場合）、ボードを接続し直します。

!!! note "Windows: CubeProgrammer でボードが認識されない"
    DFU ドライバーは STM32CubeProgrammer と一緒にインストールされます。デバイスマネージャーでボードが黄色のアイコン付きの `STM32 BOOTLOADER` として表示されている場合は、CubeProgrammer のフォルダーにあるドライバー `Drivers\DFU_Driver` を再インストールしてください。

!!! note "ボードにすでに Katapult が入っていた場合"
    Katapult は上書きされますが、これは正常です。このファームウェアはブートローダーなしでビルドされており、フラッシュの先頭から領域を使用します。

---

## 8. U1 への接続

iHeater をプリンターの USB ポートに接続し、SSH でプリンターにログインします（ユーザー名 `root`、パスワード `snapmaker`）。

```bash
ssh root@PRINTER_IP
```

!!! note "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
    プリンターのファームウェアを書き換えた後や、IP アドレスが変わった後に表示されます。古いキーを削除して、再接続してください。
    ```bash
    ssh-keygen -R PRINTER_IP
    ```

ボードの ID を調べます。

```bash
ls /dev/serial/by-id/
```

結果：

```
usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
```

---

## 9. コンフィグ

以降は通常の Klipper と同じです。[コンフィグ](configuration.md) を参照してください。

1. `iHeater.cfg` を `printer.cfg` と同じ場所に置きます（Fluidd のコンフィグエディターまたは SSH を使用）。
2. `printer.cfg` に次を追加します。
    ```ini
    [include iHeater.cfg]
    ```
3. `iHeater.cfg` にボードの ID を指定します。
    ```ini
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-if00
    ```
4. Klipper を再起動します（`FIRMWARE_RESTART`）。

!!! warning "プリンターのファームウェア更新"
    U1 のファームウェアを更新した後は、`printer.cfg` に `[include iHeater.cfg]` が残っていることを確認してください。プリンター上の Klipper のバージョンが変わった場合は、対応するコミットから iHeater のファームウェアをビルドし直してください（手順 3〜7）。

---

## トラブルシューティング

**`MCU 'iHeater' protocol error` / `Protocol error connecting to printer`**
iHeater のファームウェアのバージョンが、プリンター上の Klipper と一致していません。U1 のファームウェアバージョンに対応するコミットからビルドし直してください（手順 3）。

**ボードが `/dev/serial/by-id/` に表示されない**

- ファームウェアが `8KiB bootloader` でビルドされています。`No bootloader` でビルドし直してください。
- BOOT0 ジャンパーを外し忘れています。ボードが DFU モードのままになっています。
- USB ケーブルにデータ線がありません。
- プリンターがそもそもデバイスを認識しているか確認してください：`dmesg | tail -20`

**ビルド時に `undefined reference to 'usb_clear_stall'` が出る**
手順 4 が実行されていません。

**ビルド時に `region FLASH overflowed` が出る**
ファームウェアがフラッシュに収まりません。`Optional features` で不要な項目を無効にしてください（手順 5）。

**`gcode command M141 already registered`**
同じ名前のマクロが、プリンターの別のコンフィグですでに定義されています。どちらか一方をコメントアウトしてください。

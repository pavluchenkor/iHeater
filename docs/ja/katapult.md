# ファームウェア

このドキュメントでは、**iHeater** マイクロコントローラ向け **Katapult** ブートローダのビルドと書き込みについて説明します。Katapult を使用すると USB 経由で Klipper ファームウェアを更新でき、**iHeater** コントローラへ **Klipper** ファームウェアをインストールする際にも使用します。

---

## 要件

* STM32F042F6P6
* iHeater ボード
* 初回書き込み用の ST-Link V2 プログラマ、または USB ケーブル
* Raspberry Pi やプリンタなどの Linux システム

!!! warning "プリンタ上で直接ファームウェアをビルドして書き込めない場合"
    [WSL セクションを参照してください](user-mods/software/wsl2-ubuntu-ff/)

---

## Katapult のビルド

1. Katapult リポジトリをクローンします:

```bash
git clone https://github.com/Arksine/katapult.git
```

```bash
cd katapult
```

```bash
make menuconfig
```

2. `menuconfig` で以下を選択します:

* MCU Architecture: STM32
* Processor model: STM32F042
* Clock Reference: Internal
* Communication interface: USB (on PA9/PA10)
* Application start offset: **8KiB offset**
* [x] Support bootloader entry on rapid double click of reset button
* [x] Enable bootloader entry on button (or gpio) state
* (!PA4) Button GPIO Pin
* [x] Enable Status LED
* (PA5) Status LED GPIO Pin

![menuconfig](../img/katapult_menuconfig.jpg)

3. ファームウェアをビルドします:

```bash
make
```

ファームウェアファイルは `out/katapult.bin` に作成されます。

---

## DFU 経由で Katapult を書き込む

> この手順は、Katapult ブートローダを書き込むために一度だけ必要です。

### 準備

`dfu-util` がまだインストールされていない場合はインストールします:

```bash
sudo apt install dfu-util
```

BOOT0 ジャンパを取り付け、ボードの電源を入れ直すか RESET を押します。
マイクロコントローラが DFU モードに入ります。

接続を確認します:

```bash
lsusb
```

次のデバイスが表示されるはずです:

```text
ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```

### Katapult の書き込み

次を実行します:

```bash
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

成功時の出力例:

```text
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

ジャンパを取り外し、MODE ボタンを押したまま RESET を押して離すか、USB を再接続します。

再起動後、次を実行します:

```bash
ls /dev/serial/by-id/*
```

次のデバイスが表示されるはずです:

```bash
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

権限エラーが発生する場合:

```bash
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

### 注記

* Katapult は Flash の先頭 8 KB を使用するため、**Klipper では必ず 8KiB オフセットを指定する必要があります**。
* DFU には RESET のダブルクリック、または GPIO ボタン (PA4) で入れます。
* PA13/PA14 は SWD に使用されます。
* Katapult 書き込み後は ST-Link プログラマは不要です。以降の更新は USB 経由で実行できます。

## iHeater へのファームウェアインストール

### ファームウェアのビルド

```bash
cd klipper/
```

```bash
make menuconfig
```

#### 設定メニューで以下を選択します:

```text
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```

#### 未使用のオプションをすべて無効にします:

```text
[*] Support micro-controller based ADC (analog to digital)
[ ] Support communicating with external chips via SPI bus
[ ] Support communicating with external chips via I2C bus
[*] Support GPIO based button reading
[ ] Support Trinamic stepper motor driver UART communication
[ ] Support 'neopixel' type LED control
[ ] Support measuring fan tachometer GPIO pins
    *** LCD chips ***
[ ] Support ST7920 LCD display
[ ] Support HD44780 LCD display
    *** External ADC type chips ***
[ ] Support HX711 and HX717 ADC chips
```

#### 設定を保存してメニューを終了します。

#### ファームウェアをビルドします

```bash
make clean
```

```bash
make
```

!!! note "結果として次のファイルが作成されます:"

```text
Creating hex file out/klipper.bin
```

### iHeater ボードへのファームウェアインストール

!!! note "必要に応じて python3-serial をインストールしてください"

```bash
sudo apt install python3-serial
```

**以降の手順は、Katapult ブートローダがすでにインストール済みであることを前提としています。**

* ファームウェア書き込みモードで iHeater をホストに接続します: USB 接続時に MODE を押し続けるか、RESET を 2 回押します。

* デバイスを探します:

```bash
ls /dev/serial/by-id/
```

次のようなものが表示されるはずです:

```text
usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
```

* 必要に応じて `flashtool` をインストールします:

```bash
pip install flashtool
```

* デバイス ID を自分のものに置き換えて実行します:

```bash
python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin
```

想定される出力:

```text
Flashing '/home/pi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 20 pages

Verifying (block count = 319)...

[##################################################]

Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18

Flash Success
```

* 確認します:

```bash
ls /dev/serial/by-id/
```

出力はおおよそ次のようになります:

```text
usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00
```

`iHeater は Klipper で使用できる状態です`

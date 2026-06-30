#  ファームウェア書き込み

このドキュメントでは、**iHeater** マイクロコントローラー用の **Katapult** ブートローダーのビルドと書き込み手順を説明します。Katapult ブートローダーを使用すると USB 経由で Klipper ファームウェアを書き込めます。また、**iHeater** コントローラーへ **Klipper** ファームウェアをインストールする手順も含まれています。

---

## 必要なもの

- STM32F042F6P6
- iHeater ボード
- USB ケーブル
- Linux システム（例: Raspberry Pi またはプリンター）

!!! warning "プリンター上でファームウェアをビルドして書き込めない場合"
    [WSL のセクションを参照してください](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

---

## Katapult のビルド

1. Katapult リポジトリをクローンします:

```bash
git clone https://github.com/Arksine/katapult.git
```
```
cd katapult
```
```
make menuconfig
```

2. `menuconfig` で選択します:

![menuconfig](../../img/katapult_menuconfig.jpg)

3. ビルド:

```bash
make
```

ファームウェアは `out/katapult.bin` に作成されます。


---

## DFU 経由で Katapult を書き込む

> この手順は Katapult 自体を書き込むために、最初に 1 回だけ必要です。

### 準備:
dfu-util ユーティリティがまだインストールされていない場合はインストールします:
    
    sudo apt install dfu-util

ボードのバージョンに応じて:

=== "r1"

    BOOT0 にジャンパーを取り付け、ボードの電源を入れ直します（または RESET ボタンを押します）。
    マイクロコントローラーが DFU モードで起動します。

=== "r1.1"

    BOOT ボタンを押したままボードの電源を入れ直し（または RESET ボタンを押し）、BOOT を離します。
    マイクロコントローラーが DFU モードで起動します。

接続を確認します:

    lsusb

結果:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Katapult の書き込み:
DFU モードに切り替えます。

コマンドを実行します:

```
dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave
```

書き込み成功例:

```
Downloading to address = 0x08000000, size = 4968
Download        [=========================] 100%         4968 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

DFU モードを終了します。

再起動後:
```
ls /dev/serial/by-id/*
```
結果:
```
/dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
```

権限がない場合、書き込み時にエラーが発生することがあります。アクセス権を付与するには次のコマンドを実行します:
```
sudo chmod 777 /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00
``` 

!!! warning "問題が発生した場合"
    MCU のメモリを消去して、前の手順を繰り返してください。
    ```
    touch /tmp/empty.bin
    ```
    ```
    dfu-util -a 0 -d 0483:df11 -s :mass-erase:force -D /tmp/empty.bin
    ```

### 注意事項

- Katapult は Flash の先頭 8 KB を使用するため、**Klipper では必ず 8 KiB のオフセットを指定してください**。
- DFU に入るには、ダブル Reset または GPIO（PA4）のボタンを使用できます。
- PA13/PA14 が SWD に使用されている場合。
- Katapult の書き込み後は ST-Link を使用する必要はありません。以降の作業はすべて USB 経由で行えます。

## iHeater へのファームウェアのインストール

### ファームウェアのコンパイル
```
cd ~/klipper
```
```
make menuconfig
```

#### 設定メニューで選択する項目
```
Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F042)

Bootloader offset (8KiB bootloader)

Clock Reference (Internal clock)

Communication interface (USB (on PA9/PA10))
```
#### 不要な項目をすべて無効にする
```
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

#### 保存してメニューを終了します。

#### ファームウェアをコンパイルします
```
make clean
```
```
make
```

結果:

    Creating hex file out/klipper.bin

### iHeater ボードへのファームウェアの書き込み

!!! note "python3-serial のインストールが必要な場合があります"
    
    sudo apt install python3-serial

**以下では、Katapult ブートローダーがインストール済みの場合の手順を説明します**

- iHeater をプログラミングモードでホストに接続します（接続時に Mode ボタンを押し続ける、または RESET を 2 回押します）。

- 検索を実行します:
    ```
    ls /dev/serial/by-id/
    ```
    結果は次のようになります:

    ```
    usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX
    ```

    - 必要に応じて flashtool をインストールします

    ```
    pip install flashtool
    ```

- 自分の ID に変更して、次を入力します:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXXXXXXXXXXXX-XXXX -f ~/klipper/out/klipper.bin

    結果:

        Flashing '/home/pi/klipper/out/klipper.bin'...

        [##################################################]
        
        Write complete: 20 pages
        
        Verifying (block count = 319)...
        
        [##################################################]
        
        Verification Complete: SHA = 8A3DDF39A0E70B684DC6BAF74EF8F089EBDD6C18
        
        Flash Success

- 確認します:
    ```        
    ls /dev/serial/by-id/
    ```
    結果:

        usb-Klipper_stm32f042x6_0C0018000D53304347373020-if00

    ```iHeater は Klipper で使用する準備ができています```


# Klipper設定

このページでは、iHeaterの設定ファイルのインストールとKlipperで使用するための設定について説明します。

## 要件

### ハードウェア
  - iHeaterコントロールボード
  - NTC 100K 3950 サーミスタ（2個）
  - チャンバー用 PTCヒーターエレメント 220V 100W
  - チャンバー内の空気循環用 7530 220V ファン
  - Thermal Protector KSD9700、または同等品（220 V、5 A、130 °C）

### ソフトウェア
  - Klipper（最新版）
  - 設定済みで動作しているKlipperホスト

## Klipper設定


iHeater.cfg設定ファイルをprinter.cfgファイルがあるフォルダー（/klipper_configの場合があります）にコピーし、[include]ディレクティブを使用してprinter.cfgに追加します。


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

printer.cfgを開いて、以下を追加します。

    [include iHeater.cfg]

## iHeater MCUの接続

iHeater.cfgファイルを変更し、取得したIDを指定します。

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## 使用前の準備

設定ファイルには次のセクションが含まれています。

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # チャンバーの目標温度、°C
variable_start_offset: 10           # 印刷開始に十分なチャンバー温度、°C
variable_delta_temp: 10             # チャンバー温度とヒーター温度の差、°C
variable_min_heater_temp: 50        # ヒーターの最低温度（冷却用）、°C
variable_max_heater_temp: 100       # ヒーターの最高温度、°C
variable_control_interval: 1.0      # 制御関数の呼び出し間隔、秒
variable_air_min_delta: 0.5         # チャンバーの目標温度と現在温度の最小差（ヒーター = 目標温度 + delta_temp）、°C
variable_air_max_delta: 5.0         # チャンバーの目標温度と現在温度の最大差（ヒーター = max_heater_temp）、°C
gcode:
```

**ヒーターの最大許容温度は、筐体の材質によって異なります。**

確認手順:

!. ベッド加熱を90-100°Cにします
1. FluiddまたはMainsailのインターフェイスから、ヒーター温度を100°Cに設定します。
2. iHeaterがプリンターの密閉された空間内にあることを確認します。
3. 設定温度に達した後、ヒーターが筐体のプラスチック部品に接触している箇所を確認します。プラスチックが軟化してはいけません。
4. 温度を5-10°C上げて、確認を繰り返します。
5. 筐体変形のリスクがないヒーターの最大許容温度に達するまで繰り返します。

この方法により、安全な最高温度を判断し、iHeaterの性能を最大限に引き出すことができます。


## 使用方法

### チャンバー加熱の制御コマンド
- チャンバー温度の設定:
 

        M141 S60  ; チャンバー温度を60°Cに設定します

- 温度到達の待機:

        M191 S60  ; チャンバー温度が60°Cに達するまで待機します

- チャンバー加熱の停止:

        iHEATER_OFF   ; チャンバー加熱をオフにします

- スライサーの終了G-codeに`iHEATER_OFF`を追加して、チャンバー加熱を正しくオフにします。

### スタートg-code

最近のスライサーは、印刷用g-codeの生成時にアクティブチャンバーを自動的に有効化できます。そのためには、フィラメントのプロパティでチャンバー温度を指定する必要があります。スライサーにこの機能がない場合は、スタートg-codeにアクティブチャンバー加熱を有効化するコマンドを追加する必要があります。

手順:

- チャンバーの目標温度を設定する
- チャンバーを効率的かつ素早く加熱するため、ベッド加熱を有効にする
- 標準の印刷スタートG-codeを続行する

スタートg-codeの例
```
; --- スタートG-codeの開始 ---

; ****** iHeater開始 ******
M141 S60       ; チャンバー温度を60°Cに設定
; ****** iHeaterブロック終了 ******

; --- その他のスタートg-code ---
; ベッド加熱の有効化
...
```
!!! warning "iHeater制御マクロを正しく終了するには、プリンターの終了g-codeにiHEATER_OFFコマンドを追加する必要があります"

```
; --- 終了g-codeの開始 ---

; ****** iHeaterブロック開始 ******
iHEATER_OFF
; ****** iHeaterブロック終了 ******

; --- その他の終了g-code ---
...
```
## 無効化

iHeaterを無効化するには、printer.cfgファイル内の[include iHeater.cfg]行をコメントアウトします。
```
# [include iHeater.cfg]
```

また、スタートg-codeと終了g-codeから該当する行を削除します。

## 注意事項
- 安全:

    - すべての接続が正しく安全に行われていることを確認してください。
    - min_tempとmax_tempの値が機器の仕様に合っていることを確認してください。

- 機器の確認:
    - 使用前にヒーターとファンの動作をテストしてください。
    - 初回起動時は温度を監視してください。
- PID設定:
    - 必要に応じて、正確な温度制御のためにPIDキャリブレーションを実行してください。

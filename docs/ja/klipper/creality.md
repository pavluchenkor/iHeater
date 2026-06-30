## Creality Helper Script を使用した Creality プリンター向け iHeater ファームウェア

iHeater のファームウェア導入と連携を正常に行うには、以下の手順に従ってください。

### 1. Creality Helper Script をインストールする

Creality Helper Script プロジェクトのドキュメントページを開き、スクリプトのインストール手順に従ってください。

**リソース:**

* 動画ガイド: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* テキスト手順: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. プリンターの root アクセスとファイルシステムへのアクセスを取得する

スクリプトにより Mainsail へのアクセスと、ファームウェア設定ファイルへのアクセスが有効になります。インストールが正常に完了したら、ブラウザーからプリンターのインターフェースにアクセスでき、設定ファイルへアクセスできることを確認してください。

### 3. 既存の `fan-control.cfg` を削除する

Helper Script を使用した Creality プリンターでは、デフォルトで `M141` と `M191` マクロを含む `fan-control.cfg` ファイルがすでに作成されている場合があります。これは iHeater 設定内の同様のマクロと競合します。

ファイル名を変更してください:

```
/usr/data/printer_data/config/fan-control.cfg
```
を fan-control.cfg.bak に変更します

### 4. 新しい `fan-control.cfg` をコピーする

マクロおよびチャンバー温度制御ロジックと互換性のあるバージョンの [fan-control.cfg](../../../printers/creality/config/fans-control.cfg) に置き換えてください。

新しいファイルを同じフォルダーに配置します:

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. iHeater 設定を追加する

`iheater.cfg` ファイルを同じディレクトリにコピーします:

```
/usr/data/printer_data/config/iheater.cfg
```

次に `printer.cfg` を開き、ファイルの末尾に次の行を追加します:

```ini
[include iheater.cfg]
```

---

続いて、サーミスター、ヒーター、動作モード、マクロなど、iHeater の設定手順に従ってください。

!!! warning "プリンター上でファームウェアをビルドして書き込めない場合"
    [WSL のセクションを参照してください](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)

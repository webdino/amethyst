# Project Amethyst

WebViewer for RZ/G (patches for Firefox/Gecko)

## 仕様と設計

Gecko エンジンを使い、HTML ページを読み込み表示するためのシンプル・軽量・不要な通信のない WebViewer (HTML ビューアクライアント) です。

- 制限事項: https://github.com/webdino/amethyst/wiki/Limitations
- リモートデバッグ手順: https://github.com/webdino/amethyst/wiki/Remote-Debug

## パッチファイルの解説

各パッチファイルで実装する機能の設計および Firefox のソースツリーへの適用手順を説明します。

### webviewer.patch

WebViewer 本体です。Firefox ブラウザのメインウィンドウを実装する browser.xul を置き換えます。
このパッチにより、Firefox オリジナルの browser/base/content/browser/browser.xul を利用せず、代わりに browser/amethyst/content/browser/browser.xul をデフォルトのメインウィンドウとして読み込み起動するようになります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/webviewer.patch
```

メインウィンドウ以外のコードを残すことで、必要に応じて `about:config` や `about:preferences` などの画面を使って設定変更を行ったり、ブラウザが搭載する各種の機能を温存することが出来ます。但し、メインウィンドウの実装に依存するような機能や WebAPI は (そのままでは) すべて動作しなくなります。

### enable-webrtc.patch

WebRTC を有効にするパッチです。ポップアップ通知を表示することなく、自動でカメラやマイクなどのデバイスの使用を許可します。
すべてのサイトで自動的に許可するわけではなく、必要なサイトの許可をあらかじめ設定することで動作します。設定内容を以下に例示します。

```
プロファイルフォルダ内のpermissions.sqlite
moz_perms テーブル
id=1
origin='https://media-example.kou029w.now.sh'
type='camera'
permission=1
expireType=0
expireTime=0
modificationTime=1561519336884 -- Date.now()
```

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/enable-webrtc.patch
```

### enable-form-validation.patch

フォームバリデーションを有効にするパッチです。通常の Firefox と同様に、form の submit イベントに介入し、フォームに入力された内容が要件を満たすかどうかを判定、満たされない項目にエラーを示すポップアップを表示するようになります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/enable-form-validation.patch
```

### disable-crashreporter.patch

ビルドオプションに関係なくクラッシュレポートの送信を強制的に無効にするパッチです。送信モジュールがビルドされなくなり、applicaiton.ini の設定や環境変数 MOZ_CRASHREPORTER の値も無視されるようになります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-crashreporter.patch
```

### disable-updater.patch

ビルドオプションに関係なくブラウザー本体の自動更新機能を無効にするパッチです。更新モジュールがビルドされなくなるほか、about:support の更新履歴の行も表示されなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-updater.patch
```

### disable-addon-autoupdate.patch

拡張機能の自動更新とメタデータの更新、ブロックリストの更新を無効にするパッチです。また、about:addons を開いたときにおすすめのアドオンも表示されなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-addon-autoupdate.patch
```

### disable-telemetry.patch

Telemetry と Studies を無効にするパッチです。設定値に関係なくデータの収集と送信が行われなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-telemetry.patch
```

### disable-webpush.patch

Web Push API を無効にするパッチです。設定値に関係なく mozilla のサーバーを介したプッシュ通信が行われなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-webpush.patch
```

### disable-wifigeo.patch

Geolocation API を無効にするパッチです。設定値に関係なく mozilla または google のサーバーを介した、接続ネットワークによる位置情報の取得が行われなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-wifigeo.patch
```

### disable-captivedetect.patch

Captive Portal を検知する機能を無効にするパッチです。通常は一定間隔で mozilla の特定のサーバーに通信して Captive Portal を検知していますが、この通信を停止させます。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-captivedetect.patch
```

### disable-ocsp.patch

OCSP による証明書の自動更新を無効にするパッチです。通常は一定間隔で OCSP プロトコルで認証局の各サーバーにアクセスして、更新可能な証明書を確認していますが、設定値に関係なく確認と更新を行われないようになります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-ocsp.patch
```

### disable-safebrowsing.patch

セーフブラウジングに関する機能を無効にするパッチです。mozilla または google へのアクセス URL の確認、ダウンロードファイルの確認、フィッシング詐欺対策とマルウェア保護リストの更新、トラッキング保護リストの更新が、設定値に関係なく行われないようになります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-safebrowsing.patch
```

### disable-searchengine-update.patch

検索エンジンの自動更新を無効にするパッチです。通常は初回起動時と一定間隔で、地域に応じた検索エンジンを選定し、検索エンジンプラグインを最新にするための通信を行いますが、設定値に関係なく行われないようになります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-searchengine-update.patch
```

### disable-snippets.patch

about:home に表示されるスニペットの取得を無効にするパッチです。設定値に関係なく、スニペットの取得失敗となります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-snippets.patch
```

### disable-homepage-override.patch

ブラウザーの更新情報の自動表示を無効にするパッチです。通常は初回起動時やブラウザーの更新時に更新内容を知らせるウェブページを新規タブに自動的に表示しますが、設定値に関係なくウェブページが表示されなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-homepage-override.patch
```

### disable-attribution-code.patch

mozilla のマーケティングを測定および支援を無効にするパッチです。インストール時に利用端末に配置されたキャンペーンおよび参照データの読み込みが行われなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-attribution-code.patch
```

### disable-fxaccounts.patch

Firefox Account と Firefox Sync を無効にするパッチです。設定値に関係なく常に無効と判断され、about:preferences#sync が表示されなくなるほか、タブや URL バー、コンテキストメニューなどから、Firefox Account と Firefox Sync に関連する項目が表示されなくなります。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/disable-fxaccounts.patch
```

### removed-files.patch

これらの無効化された機能をビルドから排除し、ビルド工程の短縮と、バイナリサイズとメモリフットプリントの減少に寄与します。
なお、無効化されたすべての機能に関するモジュールが排除されるわけではありません。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/removed-files.patch
```

### about-dialog.patch

「Firefox について」のダイアログに対するパッチです。下記のブランディング変更に伴うレイアウト調整が行われています。
但し、WebViewer ではこのダイアログを開く手段を断っているため、実際にこのダイアログを目にすることはないと思われます。

以下のコマンドで Firefox のソースツリーにパッチを当てます。

```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/FIREFOX_60_1_0esr_RELEASE/about-dialog.patch
```

### browser/amethyst/branding

ブランディングに関するファイル群です。通常の Firefox ブランディングファイルと差し替える形で利用します。
画像ファイルなどが含まれるため、パッチ形式ではなくソースツリーをそのまま置いてあります。

以下のコマンドで Firefox のソースツリーにコピーします。

```
mkdir -p /path/to/mozilla-esr60/browser/amethyst
cp -rp /path/to/amethyst/browser/amethyst/branding /path/to/mozilla-esr60/browser/amethyst/
```

ビルド前にビルド設定ファイル `.mozconfig` に以下の行を追加します。

```
ac_add_options --with-branding=browser/amethyst/branding
```

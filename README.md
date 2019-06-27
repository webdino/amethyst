# Project Amethyst
Web/HTML Viewer for RZ/G (patches for Firefox/Gecko)

# パッチファイルの解説
各パッチファイルで実装する機能の設計および Firefox のソースツリーへの適用手順を説明します。

### webviewer.patch
WebViewer 本体です。Firefox ブラウザのメインウィンドウを実装する browser.xul を置き換えます。  
このパッチにより、Firefox オリジナルの browser/base/content/browser/browser.xul を利用せず、代わりに browser/amethyst/content/browser/browser.xul をデフォルトのメインウィンドウとして読み込み起動するようになります。  

以下のコマンドで Firefox のソースツリーにパッチを当てます。
```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/webviewer.patch
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
patch -p1 -i /path/to/amethyst/enable-webrtc.patch
```

### about-dialog.patch
「Firefox について」のダイアログに対するパッチです。下記のブランディング変更に伴うレイアウト調整が行われています。  
但し、WebViewer ではこのダイアログを開く手段を断っているため、実際にこのダイアログを目にすることはないと思われます。  

以下のコマンドで Firefox のソースツリーにパッチを当てます。
```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/about-dialog.patch
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

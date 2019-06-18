# amethyst
Web/HTML Viewer for RZ/G (patches for Firefox/Gecko)

# パッチファイルの解説
### webviewer.diff
WebViewer 本体です。Firefox の browser.xul を置き換えます。  
このパッチにより、Firefox オリジナルの browser/base/content/browser/browser.xul がスキップされ、代わりに browser/amethyst/content/browser/browser.xul がビルドされ、既定の chrome ウィンドウとして実行されるようになります。  
以下のコマンドで Firefox のソースツリーにパッチを当てます。
```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/webviewer.diff
```

### about-dialog.diff
「Firefox について」のダイアログに対するパッチです。下記のブランディング変更に伴うレイアウト調整が行われています。  
が、WebViewer ではこのダイアログを開く手段を断っているため、実際にこのダイアログを目にすることはないと思われます。  
以下のコマンドで Firefox のソースツリーにパッチを当てます。
```
cd /path/to/mozilla-esr60
patch -p1 -i /path/to/amethyst/about-dialog.diff
```

### browser/amethyst/branding
ブランディングに関するファイル群です。  
画像ファイルなどが含まれるため、パッチ形式ではなくソースツリーをそのまま置いてあります。  
以下のコマンドで Firefox のソースツリーにコピーします。
```
mkdir -p /path/to/mozilla-esr60/browser/amethyst
cp -rp /path/to/amethyst/browser/amethyst/branding /path/to/mozilla-esr60/browser/amethyst/
```
ビルド前に以下の行を .mozconfig に追加します。
```
ac_add_options --with-branding=browser/amethyst/branding
```

# 外部への情報送信機能の無効化

通常の Firefox には個人情報や統計データ、クラッシュレポートなどを外部に送信する機能がある。
WebViewer for RZ/G では、そういった外部への情報送信機能を無効化して、プライバシーへの配慮と軽量化を図っている。
本文書では WebViewer for RZ/G での情報送信機能の無効化とそれを確認するための手順を記す。

## Telemetry portal へのデータ送信の無効化 [#11](https://github.com/webdino/amethyst/issues/11)

https://telemetry.mozilla.org/ へのパフォーマンス情報の送信をしない。
また、いくつかの内部向けページを削除してある。

- 送信されるパフォーマンス情報の内容を確認するページ about:telemetry
- 参加している調査の内容を確認するためのページ about:studies

### 確認内容

引数に about:blank を指定し起動後、一定時間が経っても incoming.telemetry.mozilla.org や telemetry-experiment.cdn.mozilla.net などへの通信が無い。

ログを取得する際に使用したコマンド:

```log
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# timeout -t 600 sh -c 'webviewer about:blank 2>webviewer-telemetry-sample.log'
```

アクセスする回数を計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' webviewer-telemetry-sample.log | wc -l
```

結果の一覧:

| 環境             | 結果 | 実際のログファイル                                               |
| ---------------- | ---- | ---------------------------------------------------------------- |
| RG/G2E WebViewer 60 | 0    | [webviewer-telemetry-sample.log](60/webviewer-telemetry-sample.log) |
| RG/G2E WebViewer 78 | 0    | [webviewer-telemetry-sample.log](78/webviewer-telemetry-sample.log) |

## 初回起動時のデータ取得・送信の無効化 [#12](https://github.com/webdino/amethyst/issues/12)

初回起動時に外部への通信がない。

### 確認内容

引数に内部向けページのアドレス about:blank を与えて初回起動し、その時の HTTP 通信ログの内容を通常の Firefox での結果と比較した。

ログを取得する際に実際に使用したコマンド:

```log
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# webviewer about:blank 2>firstrun.log
```

アクセスする回数を計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' firstrun.log | wc -l
```

結果の一覧:

| 環境                  | 結果 | アクセス先                                                                                                                                                                                                                                                                                                                                             | 実際のログファイル                               |
| --------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| RZ/G2E Firefox 60.1.0 | 56   | classify-client.services.mozilla.com content-signature-2.cdn.mozilla.net detectportal.firefox.com location.services.mozilla.com mozilla.org normandy.cdn.mozilla.net ocsp.digicert.com search.services.mozilla.com shavar.services.mozilla.com snippets.cdn.mozilla.net tiles.services.mozilla.com tracking-protection.cdn.mozilla.net www.mozilla.org | [firefox-firstrun.log](60/firefox-firstrun.log)     |
| RG/G2E webviewer 60    | 0    |                                                                                                                                                                                                                                                                                                                                                        | [webviewer-firstrun.log](60/webviewer-firstrun.log) |
| RG/G2E webviewer 78    | 0    |                                                                                                                                                                                                                                                                                                                                                        | [webviewer-firstrun.log](78/webviewer-firstrun.log) |

## セキュリティ強化のための通信の無効化 [#14](https://github.com/webdino/amethyst/issues/14)

前回起動時から一定時間経過後起動時に通信が無い。

### 確認内容

date コマンドにて Linux のウォールクロックを 1 年後のものに変更後、about:blank を指定し再び起動した際、外部へのリクエストが発生しない。

ログを取得する際に実際に使用したコマンド:

```log
# date -s '2019/09/20 00:00:00'
# webviewer about:blank
^C
# date -s '2020/09/20 00:00:00'
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# webviewer about:blank 2>1yearlater.log
^C
```

アクセスする回数を計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' 1yearlater.log | wc -l
```

結果の一覧:

| 環境                  | 結果 | アクセス先                                                                                                                                                                                 | 実際のログファイル                               |
| --------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| RZ/G2E Firefox 60.1.0 | 11   | detectportal.firefox.com normandy.cdn.mozilla.net ocsp.digicert.com search.services.mozilla.com shavar.services.mozilla.com tiles.services.mozilla.com tracking-protection.cdn.mozilla.net | [firefox-1yearlater.log](60/firefox-1yearlater.log) |
| RG/G2E webviewer 60     | 0   |  | [webviewer-1yearlater.log](60/webviewer-1yearlater.log)                                                                                                                                       |
| RG/G2E webviewer 78     | 0   |  | [webviewer-1yearlater.log](78/webviewer-1yearlater.log)                                                                                                                                       |

## マルウェアチェック機能の無効化 [#42](https://github.com/webdino/amethyst/issues/42)

Google や Mozilla との通信を伴うセーフブラウジング機能が無効。

### 確認内容

セーフブラウジングを有効化したプロファイルの作成し、Web ページからファイルをダウンロードしたとき、ファイルのホスト以外外部との通信が無い。

https://file-examples.com/wp-content/uploads/2017/10/file-sample_150kB.pdf にアクセス

セーフブラウジングを有効化したプロファイルの作成:

- browser.safebrowsing.phishing.enabled
- browser.safebrowsing.malware.enabled
- browser.safebrowsing.downloads.enabled
- browser.safebrowsing.downloads.remote.enabled
- browser.safebrowsing.passwords.enabled
- browser.safebrowsing.blockedURIs.enabled
- plugins.flashBlock.enabled

いずれも about:config から true に設定

- browser.safebrowsing.provider.google4.nextupdatetime
- browser.safebrowsing.provider.mozilla.nextupdatetime

どちらも about:config から "0" に設定

ログファイルを記録するために使用したコマンド:

```sh
NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5 webviewer https://file-examples.com/wp-content/uploads/2017/10/file-sample_150kB.pdf 2>malware.log
```

ログファイルから計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' malware.log | wc -l
```

| 結果 | アクセス先                                           | 実際のログファイル                             |
| ---- | ---------------------------------------------------- | ---------------------------------------------- |
| 1    | file-examples.com (ダウンロードするファイルのホスト) | [webviewer-malware.log](60/webviewer-malware.log) |
| 3    | file-examples.com (ダウンロードするファイルのホスト) | [webviewer-malware.log](78/webviewer-malware.log) |

## クラッシュレポートの無効化 [#15](https://github.com/webdino/amethyst/issues/15)

Mozilla Crash Reporter が起動しない。

### 確認内容

crashreporter のバイナリが /usr/lib64/webviewer ディレクトリ以下に存在しないことと、実際にクラッシュさせ Mozilla Crash Reporter が起動しない。

クラッシュさせる方法:

1. リモートデバッガでブラウザコンソールに接続
   - リモートデバッガで接続する方法: https://github.com/webdino/amethyst/wiki/Remote-Debug
2. chrome://browser/content/browser.xul のコンテクストで Components.utils.crashIfNotInAutomation() を実行

## Mozilla のサービス連携の無効化 [#18](https://github.com/webdino/amethyst/issues/18)

Firefox アカウント、Pocket への通信がない。
また、Firefox Sync の設定項目 about:preferences#sync にアクセスすることが出来ない。

### 確認内容

1. Linux (x86_64) PC 環境で Firefox 60 をインストール
2. Firefox を起動、Firefox アカウント、Pocket にログインしたユーザープロファイルを作成
3. 実機の rootfs パーティションの /home/root/.mozilla ディレクトリ以下にそのプロファイルをコピーしログを取得
4. about:blank、about:preferences#sync にアクセスして、外部との通信が発生しないことを確認する

通常の Firefox でそれぞれの機能を有効化したプロファイルを作成し、WebViewer のプロファイルとして読み込み、内部向けページ about:blank、about:preferences を開き、その時の HTTP 通信ログの内容を比較調査した。

ログを取得する際に実際に使用したコマンド:

```log
# rm -r .mozilla/firefox/myyqvz5n.default/
# cp -a 4s2uzw47.fx60 .mozilla/firefox/myyqvz5n.default
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# webviewer about:blank 2>blank.log
^C
# webviewer about:preferences 2>preferences.log
^C
```

アクセスする回数を計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' /path/to/logfile | wc -l
```

結果の一覧:

| 環境                      | 開いたページ      | 結果 | アクセス先                                                                                                                                                                                                                                                                                  | 実際のログファイル                                                                       |
| ------------------------- | ----------------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Linux PC                  | about:blank       | 10   | api.getpocket.com assets-cdn.github.com detectportal.firefox.com firefoxusercontent.com ocsp.digicert.com ocsp.sca1b.amazontrust.com profile.accounts.firefox.com push.services.mozilla.com tiles.services.mozilla.com                                                                      |
| RZ/G2E webviewer 8746d4c5 | about:blank       | 0    |                                                                                                                                                                                                                                                                                             | [webviewer-mozilla-services-blank.log](60/webviewer-mozilla-services-blank.log)             |
| RZ/G2E webviewer 78 | about:blank       | 0    |                                                                                                                                                                                                                                                                                             | [webviewer-mozilla-services-blank.log](78/webviewer-mozilla-services-blank.log)             |
| Linux PC                  | about:preferences | 14   | api.accounts.firefox.com api.getpocket.com assets-cdn.github.com aus5.mozilla.org detectportal.firefox.com firefoxusercontent.com ocsp.digicert.com ocsp.sca1b.amazontrust.com profile.accounts.firefox.com push.services.mozilla.com tiles.services.mozilla.com token.services.mozilla.com |
| RZ/G2E webviewer 8746d4c5 | about:preferences | 0    |                                                                                                                                                                                                                                                                                             | [webviewer-mozilla-services-preferences.log](60/webviewer-mozilla-services-preferences.log) |
| RZ/G2E webviewer 78 | about:preferences | 1    | content-signature-2.cdn.mozilla.net                                                                                                                                                                                                                                                                                            | [webviewer-mozilla-services-preferences.log](78/webviewer-mozilla-services-preferences.log) |

## Geolocation API の無効化 [#19](https://github.com/webdino/amethyst/issues/19)

Geolocation API が利用できない。

### 確認内容

Geolocation API を使用するサンプルページにアクセスし、通常の動作が無効化されて利用できないことを確認。
またその時の HTTP 通信ログにホスト googleapis.com などへのリクエストが発生しない。

サンプルページ: https://ypvfr.csb.app

ログ:

```log
root@ek874:~# NSPR_LOG_MODULES=nsHttp:3 webviewer 'https://ypvfr.csb.app/' > ns-http.log 2>&1
^C
root@ek874:~# grep 'Host' ns-http.log
[5303:Main Thread]: I/nsHttp   Host: ypvfr.csb.app
```

## Web Push API の無効化 [#20](https://github.com/webdino/amethyst/issues/20)

Mozilla の Push Service へのアクセスが発生しない。

### 確認内容

Web Push API を実行するサンプルページにアクセスし、その時の HTTP 通信ログに Push Service のホスト push.services.mozilla.com が存在しない。

サンプル: https://gauntface.github.io/simple-push-demo/

ログ:

```log
root@ek874:~# NSPR_LOG_MODULES=nsHttp:3 webviewer https://gauntface.github.io/simple-push-demo/ > ns-http.log 2>&1
^C
root@ek874:~# grep 'Host' ns-http.log
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: gauntface.github.io
[5076:Main Thread]: I/nsHttp   Host: www.google-analytics.com
```

## アドオンマネージャのデータ送信の無効化 [#21](https://github.com/webdino/amethyst/issues/21)

about:addons にアクセスして、おすすめアドオンの表示など通信が無い。

### 確認内容

about:addons にアクセスした際、通信ログにホスト discovery.addons.mozilla.org などが存在しない。

ログを取得する際に実際に使用したコマンド:

```log
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# webviewer about:addons 2>addons.log
```

アクセスする回数を計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' addons.log | wc -l
```

結果一覧:

| 環境                              | 結果 | アクセス先                                                                                                                                                                                                                                                                                                                                                                     | 実際のログファイル                           |
| --------------------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- |
| Linux (x86_64) PC 環境 Firefox 60 | 38   | addons-discovery.cdn.mozilla.net addons.cdn.mozilla.net api.getpocket.com assets-cdn.github.com detectportal.firefox.com discovery.addons.mozilla.org firefoxusercontent.com ocsp.digicert.com ocsp.pki.goog ocsp.sca1b.amazontrust.com profile.accounts.firefox.com push.services.mozilla.com safebrowsing.googleapis.com tiles.services.mozilla.com www.google-analytics.com |
| RZ/G2E webviewer 8746d4c5         | 0    |                                                                                                                                                                                                                                                                                                                                                                                | [webviewer-addons.log](60/webviewer-addons.log) |
| RZ/G2E webviewer 78         | 0    |                                                                                                                                                                                                                                                                                                                                                                                | [webviewer-addons.log](78/webviewer-addons.log) |

## Captive Portal の無効化 [#41](https://github.com/webdino/amethyst/issues/41)

http://detectportal.firefox.com/success.txt へのアクセスが発生しない。

### 確認内容

引数に内部向けページのアドレス about:blank を与えて初回起動し、その時の HTTP 通信ログにホスト detectportal.firefox.com が存在しない。

ログを取得する際に実際に使用したコマンド:

```log
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# webviewer about:blank 2>firstrun.log
```

アクセスする回数を計測するために使用したコマンド:

```sh
grep -E 'Host:[[:space:]]+[^[:space:]]+' firstrun.log | wc -l
```

結果の一覧:

| 環境             | 結果 | アクセス先 | 実際のログファイル                               |
| ---------------- | ---- | ---------- | ------------------------------------------------ |
| RG/G2E webviewer 60 | 0    |            | [webviewer-firstrun.log](60/webviewer-firstrun.log) |
| RG/G2E webviewer 78| 0    |            | [webviewer-firstrun.log](78/webviewer-firstrun.log) |

## OCSPの無効化

http://ocsp.digicert.com へのアクセスが発生しない。

### 確認内容

引数にOCSPによる証明書確認を行っているサイトを与えて起動し、その時の HTTP 通信ログにホスト ocsp.digicert.com が存在しない。

今回は https://github.com を使用する

ログを取得する際に実際に使用したコマンド:
```log
# export NSPR_LOG_MODULES=nsHttp:5,nsSocketTransport:5,nsHostResolver:5
# webviewer https://github.com 2>webviewer-ocsp.log
```
アクセスする回数を計測するために使用したコマンド:

```sh
grep -E  http://ocsp.digicert.com  webviewer-ocsp.log | wc -l
```
結果の一覧:

| 環境                  | 結果 | アクセス先                                                                                                                                                                                                                                                                                                                                             | 実際のログファイル                               |
| --------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| Linux PC | 30   | http://ocsp.digicert.com  |  [firefox-ocsp.log](78/firefox-ocsp.log)   |
| RG/G2E webviewer 78 | 0    |                           |  [webviewer-ocsp.log](78/webviewer-ocsp.log) |
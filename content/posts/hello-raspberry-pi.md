---
title: "Hello Raspberry Pi"
date: 2021-10-29T22:18:38+09:00
draft: false
categories:
  - Tech
tags:
  - Raspberry-Pi

---

desciription
## ラズパイで遊ばせてもらった！

感想をひとことでいうと、めちゃくちゃ簡単だし楽しい。もっと遊んでみたくなった。
ラズパイ欲しい😍
正直、LEDが点滅するだけでこんなにワクワクすると思ってなかった。
前から楽しそうだな〜とは思ってたけど、もっと早くに触っておけばよかった。
せっかくなのでやったことを残しておく。（もっと写真撮ればよかった🥲）

<!--more-->

## ラズパイにOSをインストールしてセットアップ

今回は事前にOSを落としてきたmicroSDを用意してもらっていたので本当にものの数分でセットアップが完了した。

- OSをダウンロードしたmicroSDを用意
- ラズパイにmicroSDをセットして、モニタとキーボード繋いで、電源コードを繋ぐ
- モニタの左上にラズパイのロゴが出てきたら起動が始まった証拠、しばしまつ
- GUI上でパスワードやタイムゾーン・言語などを聞かれるので、選択してセットアップ
- WiFiを設定する

以上。

これが終わったら、ラズパイにssh接続するためにIPを`ifconfig`コマンドで確認して、同じネットワーク内にいるPCからsshで接続したら完了。
かーんたーん！！！😎

## Node-REDをインストールする

Node-REDは、Node.jsが動く環境で動作するソフトウェアで、ラズパイでよく使われるGUI上でプログラミングできるツールである。
> Node-RED（ノード・レッド）は、IoTの一部としてハードウェアデバイス、API、オンラインサービス（英語版）を相互に接続するためにもともとIBMによって開発された、ビジュアルプログラミング用のフローベースの開発ツールである  ~Wikipediaより~

これをインストールして実行すれば、ブラウザ上からプログラムが作れる。

- パッケージアップデートの確認
```
pi@raspberrypi:~ $ sudo apt update
```

- パッケージアップデート（割と時間かかる）
```
pi@raspberrypi:~ $ sudo apt full-upgrade -y
pi@raspberrypi:~ $ sudo apt clean
```

- バージョン確認
```
pi@raspberrypi:~ $ node -v
v8.11.1
pi@raspberrypi:~ $
pi@raspberrypi:~ $ uname -a
Linux raspberrypi 4.14.79-v7+ #1159 SMP Sun Nov 4 17:50:20 GMT 2018 armv7l GNU/Linux
pi@raspberrypi:~ $
pi@raspberrypi:~ $ lsb_release -a
No LSB modules are available.
Distributor ID:	Raspbian
Description:	Raspbian GNU/Linux 9.4 (stretch)
Release:	9.4
Codename:	stretch
pi@raspberrypi:~ $
```

- Node-REDインストールとアップデート（アップデートにちょっと時間がかかる）
```
pi@raspberrypi:~ $ bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
pi@raspberrypi:~ $ update-nodejs-and-nodered
```

- Node-RED起動（初回）
```
pi@raspberrypi:~ $ node-red-start


    npm is not installed, it is recommended
    to install the latest by running:
        update-nodejs-and-nodered


Start Node-RED

Once Node-RED has started, point a browser at http://ローカルIP:1880
On Pi Node-RED works better with the Firefox or Chrome browser

Use   node-red-stop                          to stop Node-RED
Use   node-red-start                         to start Node-RED again
Use   node-red-log                           to view the recent log output
Use   sudo systemctl enable nodered.service  to autostart Node-RED at every boot
Use   sudo systemctl disable nodered.service to disable autostart on boot

To find more nodes and example flows - go to http://flows.nodered.org

Starting as a systemd service.
Started Node-RED graphical event wiring tool.
29 Oct 20:35:04 - [info]
Welcome to Node-RED
===================
29 Oct 20:35:04 - [info] Node-RED バージョン: v0.20.7
29 Oct 20:35:04 - [info] Node.js  バージョン: v8.11.1
29 Oct 20:35:04 - [info] Linux 4.14.79-v7+ arm LE
29 Oct 20:35:04 - [info] パレットエディタを無効化 : npmコマンドが見つかりません
29 Oct 20:35:04 - [info] パレットノードのロード
29 Oct 20:35:12 - [warn] 不足しているノードモジュール:
29 Oct 20:35:12 - [warn]  - node-red-node-ledborg (0.0.21): ledborg
29 Oct 20:35:12 - [info] 設定からモジュールを削除します
29 Oct 20:35:12 - [info] 設定ファイル: /home/pi/.node-red/settings.js
29 Oct 20:35:12 - [info] コンテキストストア : 'default' [module=memory]
29 Oct 20:35:12 - [info] ユーザディレクトリ : /home/pi/.node-red
29 Oct 20:35:12 - [warn] プロジェクトは無効化されています : editorTheme.projects.enabled=false
29 Oct 20:35:12 - [info] フローファイル     : /home/pi/.node-red/flows_raspberrypi.json
29 Oct 20:35:12 - [info] flow ファイルを作成します
29 Oct 20:35:12 - [warn]
---------------------------------------------------------------------
フローのクレデンシャルファイルはシステム生成キーで暗号化されています。
システム生成キーを何らかの理由で失った場合、クレデンシャルファイルを
復元することはできません。その場合、ファイルを削除してクレデンシャルを
再入力しなければなりません。
設定ファイル内で 'credentialSecret' オプションを使って独自キーを設定
します。変更を次にデプロイする際、Node-REDは選択したキーを用いてクレ
デンシャルを再暗号化します。
---------------------------------------------------------------------
29 Oct 20:35:12 - [info] フローを開始します
```

### ブラウザからNode-REDにアクセスする

`http://ローカルIP:1880`にアクセスしたら、こんな感じのGUI。

{{< figure src="/images/node-red_interface.png" class="center" width="80%">}}

必要なノードを配置して、それをフロー図のように組み合わせて入出力を定義する。
functionのノードの中身は受け取った入力をJavaScriptを書くことでプログラム的な処理を行うことが可能。

{{< figure src="/images/node-red_function.png" class="center" width="80%">}}

ここら辺のリストは、全てラズパイ上の入出力のピンに対応している。
これを見ながら適切な入力のピンにセンサーを挿せばOK。簡単すぎる、天才だ。

{{< figure src="/images/node-red_list.jpg" class="center" width="80%">}}

ラズパイを上記の写真の向きで見ると、ブラウザ上のリストのピンの位置と対応している状態になる。
GUIのプログラムから0と1を渡すスイッチを作って、それをブラウザ上で操作してあげると、LEDが光ったり消えたりする。

これが巷でよく聞くLチカである。やっと会えた…！
{{< figure src="/images/node-red_LED.jpg" class="center" width="80%">}}

手順的にはここまで大して苦労していなかったけど、とにかくOSのアップデートが時間かかったのでなんか感慨深かった。
このたった一個のLEDをついたり消したりするだけで、まさかこんなに楽しめるとは。
自分でも思って見なかったので嬉しい発見だった。
またひとつ、ワクワクするものに出会えた！

Lチカだけでこんなに嬉しいなら、[こんなの](https://www.amazon.co.jp/dp/B08M5DXS2P)を動かせた日には、発狂してしまいそう。
ちょっと時間があったので、Apacheのサーバーも立ててHelloWorldしておいた。

```
pi@raspberrypi:~ $ sudo apt install apache2
```
{{< figure src="/images/raspbery-pi_apache.jpg" class="center" width="80%">}}

Hello Worldしといた。

{{< figure src="/images/raspbery-pi_hello.jpg" class="center" width="80%">}}

今日はここでタイムアップ。

距離を超音波の帰ってくる時間で計測できるセンサーなどもあったけど、OSアップデートに時間がかかってしまいそこまでできなかった。
Node-REDのGUIで確認できたノードの一覧を見た限り、かなり色々なことができそうな気配だったので、センサーをいろいろ試してみるのが楽しそう。

---
title: "Remote-ssh + EC2でLaravel開発環境"
date: 2021-05-09T11:44:20+09:00
draft: false
categories:
  - Tech
tags:
  - Laravel
  - EC2
---

業務で研修に携わることになり、ConoHa VPS上のLaravelの開発環境にRemote-SSHからアクセスして開発するのが圧倒的に簡単だった。

最初にLaravel使ったときはAWSの優待を使ってcloud9で開発してたけど、VS Codeを使って手元で開発できたほうがいいよなぁということで。EC2にインスタンス立ててRemote-SSHでローカルからアクセスしてみるというのを試してみた。

<!--more-->

## AWSにインスタンスをたてる

とりあえず何も考えずにt2.microでつくって、SSHだけアクセス元のIPを制限した。
マイIPというボタン一つで現在のアクセスIPを登録してくれるのでとりあえずシュッと作るだけなら何も考えなくてOK。

まずは特に設定変更もせずにインスタンス作成ボタンを押すと、鍵を登録するか新規作成してって言ってくれるので、新規作成にしてボタンポチで作成＆ダウンロード。

{{< figure src="/images/create_key.png" class="center" width="50%">}}

作成が完了しました、とでてきたらインスタンスの詳細画面に行って接続情報を確認。

{{< figure src="/images/connect_info.png" class="center" width="50%">}}

こんな風に、接続用コマンドまで書いてくれているので即コピペでOKな感じになってた。親切。
インスタンスが起動できたら、手元のVS Codeから接続できることを確認していく。

### Remote-sshでインスタンスに接続

- 拡張機能からRemote-sshをインストール。
- ＋ボタンから新規接続の設定へ。
{{< figure src="/images/remote_ssh.png" class="center" width="50%">}}

- `ssh -i {ダウンロードした鍵のパス} {ユーザー名}@{ホスト名}` でEnter。
{{< figure src="/images/ssh_connection.png" class="center" width="50%">}}

- 設定の保存先を聞かれるので適宜選択。`~/.ssh/config`で問題ないのでそうした。
- Remote-sshの拡張機能のメニュー内のSSH TARGETに接続先が追加されていればOK.
- ホスト名の右端のアイコンクリック、もしくはホスト名を右クリックしたら接続できる。

### Laravelの環境をつくる

- PHPインストール

```bash
$ sudo amazon-linux-extras install php7.4
$ php -v
  PHP 7.4.15 (cli) (built: Feb 11 2021 17:53:39) ( NTS )
  Copyright (c) The PHP Group
  Zend Engine v3.4.0, Copyright (c) Zend Technologies
$
```

- Laravelに必要なもろもろをインストール

```bash
$ sudo yum install php-mbstring php-pecl-memcached php-gd php-apcu php-xml
$ sudo yum install httpd
# composerインストール
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer

# Laravelをインストールする
$ cd /var/www/
$ sudo php /usr/local/bin/composer create-project laravel/laravel PROJECT_NAME --prefer-dist
$ cd PROJECT_NAME/
$ sudo chmod -R 777 storage
$ sudo chmod -R 777 bootstrap/cache
$ php artisan --version
```

- vhostの設定を行う

```bash
$ cd /etc/httpd/conf.d
$ sudo vim vhost.conf

# 今回のconfの設定
$ cat /etc/httpd/conf.d/vhost.conf
  <VirtualHost *:80>
      DocumentRoot /var/www/PROJECT_NAME/public       # Laravelの公開ディレクトリをドキュメントルートに
      ServerName ec2-*****.compute-1.amazonaws.com    # EC2インスタンス情報の「パブリックIPv4 DNS」を設定
      ServerAlias ec2-*****.compute-1.amazonaws.com
        #.htaccessを利用可能にする
        AllowOverride All
        # Laravelで利用する環境変数を development に設定
        SetEnv APP_ENV development
        #アクセス許可
        Require all granted
      </Directory>
  </VirtualHost>
   $
```

- Apacheを起動する

```bash
$ sudo systemctl status httpd  # inactiveな状態を確認
$ sudo systemctl start httpd
$ sudo systemctl status httpd  # activeな状態を確認
```

{{< figure src="/images/hello_Laravel.png" class="center" width="50%">}}

ひとまずこれで完了。あっという間にできてしまった。

#### 参考にさせていただいたサイト

- [https://qiita.com/h19e/items/02d1301d4fdd8dfa88ac](https://qiita.com/h19e/items/02d1301d4fdd8dfa88ac)
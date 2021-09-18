---
title: "lnコマンドおさらい"
date: 2021-09-18T23:55:09+09:00
draft: false
categories:
  - Tech
tags:
  - LinuC

---

宣言通り、LinuCの勉強中です。シンボリックリンクとハードリンクの挙動、毎回忘れてググってしまうので書き留めておく。
特に削除する時の挙動が曖昧になるので動作検証してちゃんと覚えたい。

<!--more-->

## シンボリックリンクとハードリンク

シンボリックリンクとハードリンクの違いの前に、Linuxにおけるファイルとは何かをおさらい。Linuxでは、ファイルをディスクに保存すると、ユニークなiノード番号が割り振られる。（ちなみにディレクトリも特殊な形のファイルである。）このiノードには、ファイルに関する様々な情報が保持されている。

例えば、ファイルのアクセス権やファイルサイズ、ファイル種別などの`ls -l`で閲覧できる情報などと、**ディスク上の物理的な保存場所を示す情報**などである。
物理的な保存場所に格納されているデータがファイルの実体であり、たとえ実体がひとつであってもその実体を参照するファイルが複数存在する、ということはありえる。これがハードリンク。

ハードリンクの場合は、元のファイルとハードリンクとして作成したファイルのどちらも同じiノードを持っていてファイルの実体がひとつ（つまり物理的な保存場所が同一）なので、この二つに区別はない。いずれかを編集すると当然両方に影響がある。
同一の実体ファイルに対して変更を加えているからである。ハードリンクが3つ、4つ、と増えても同じである。*iノードはファイルシステムごとに管理される*ため、ハードリンクはリンク元のファイルが存在する同一のファイルシステム上にしか作成できない。下記の例だと、file.originalが存在するファイルシステム上だけでこれをリンク元にしたハードコピーが作成できる。

シンボリックリンクは参照先のファイルの場所（実体ファイルの場所ではない）を示すファイル。いわゆるMacのエイリアス、Winでいうショートカット。シンボリックリンクを削除してもファイルの実体には影響がないし、逆にシンボリックリンクが残ったまま実体ファイルを削除することも可能。（ただしリンクはエラーとなる。）シンボリックリンクが保持している情報は、参照先のファイルのパスになるので別のファイルシステム上からリンクを作成することができる。

## ハードリンクについて

ハードリンク作成時のコマンドは`ln リンク元（＝実体） リンクファイル`。

### ハードリンクの挙動

ハードリンクはiノードが全て同じ。また、左から3番目の数字は参照の数を表すので、ハードリンクが増えるたびに数値が増えていることがわかる。

```bash
~ ❯❯❯ touch file.original
~ ❯❯❯ ln file.original file.hard1
~ ❯❯❯ ls -il file*
70677112 -rw-r--r--  2 mochiko  2033490572  0  9 19 00:11 file.hard1
70677112 -rw-r--r--  2 mochiko  2033490572  0  9 19 00:11 file.original
~ ❯❯❯ ln file.hard1 file.hard2
~ ❯❯❯ ls -il file*
70677112 -rw-r--r--  3 mochiko  2033490572  0  9 19 00:11 file.hard1
70677112 -rw-r--r--  3 mochiko  2033490572  0  9 19 00:11 file.hard2
70677112 -rw-r--r--  3 mochiko  2033490572  0  9 19 00:11 file.original
~ ❯❯❯
```

コピーするとiノードは異なるものになるので編集してもお互い影響を受けない。

```bash
~ ❯❯❯ cp -pi file.hard1 file.hard-copy
~ ❯❯❯ ls -il file*
70677358 -rw-r--r--  1 mochiko  2033490572  0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  3 mochiko  2033490572  0  9 19 00:11 file.hard1
70677112 -rw-r--r--  3 mochiko  2033490572  0  9 19 00:11 file.hard2
70677112 -rw-r--r--  3 mochiko  2033490572  0  9 19 00:11 file.original
~ ❯❯❯
~ ❯❯❯ echo "Hello Linux" > file.original
~ ❯❯❯ cat file.original
Hello Linux
~ ❯❯❯ cat file.hard1
Hello Linux
~ ❯❯❯ cat file.hard2
Hello Linux
~ ❯❯❯ cat file.hard-copy
~ ❯❯❯
```

### ハードリンクの削除

`rm ハードリンク` or `unlink ハードリンク`でハードリンクの削除が可能。

```bash
~ ❯❯❯ ls -li file.original file.hard*
70677358 -rw-r--r--  1 mochiko  2033490572   0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.hard1
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.hard2
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.hard3
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.original
~ ❯❯❯
~ ❯❯❯ rm file.hard1
~ ❯❯❯ ls -li file.original file.hard*
70677358 -rw-r--r--  1 mochiko  2033490572   0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.hard2
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.hard3
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:49 file.original
~ ❯❯❯
~ ❯❯❯ unlink file.hard3
~ ❯❯❯ ls -li file.original file.hard*
70677358 -rw-r--r--  1 mochiko  2033490572   0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.hard2
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.original
~ ❯❯❯
```

ハードリンクが削除されてもそのファイルへの参照が残っていればファイルの実体が消えることはない。なので、もともとハードリンクの参照元となっていた`file.original`を先に削除しても他のハードリンクには影響がない。

```bash
~ ❯❯❯ ls -li file.*
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.hard
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.original
~ ❯❯❯ unlink file.original
~ ❯❯❯ ls -li file.*
70677112 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:49 file.hard
~ ❯❯❯
```

## シンボリックリンクについて

シンボリックリンク作成時のコマンドは`ln -s リンク元（＝実体） リンクファイル`。

### シンボリックリンクの挙動

`ls -l`で確認するとファイル種別はシンボリックリンクを表す`l`となり、リンク元のファイル名も取得できている。ハードリンクに対してシンボリックリンクを作成することも可能。

```bash
~ ❯❯❯ ln -s file.original file.symlink
~ ❯❯❯ ls -il file.original file.sym*
70677112 -rw-r--r--  3 mochiko  2033490572   0  9 19 00:11 file.original
70679436 lrwxr-xr-x  1 mochiko  2033490572  13  9 19 00:31 file.symlink@ -> file.original
~ ❯❯❯
~ ❯❯❯ ln -s file.hard2 file.symlink2
~ ❯❯❯ ls -il file*
70677358 -rw-r--r--  1 mochiko  2033490572   0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:35 file.hard1
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:35 file.hard2
70677112 -rw-r--r--  3 mochiko  2033490572  12  9 19 00:35 file.original
70679436 lrwxr-xr-x  1 mochiko  2033490572  13  9 19 00:31 file.symlink@ -> file.original
70679720 lrwxr-xr-x  1 mochiko  2033490572  10  9 19 00:39 file.symlink2@ -> file.hard2
~ ❯❯❯
```

シンボリックリンクのコピーには、オプションの`-d`が必要。オプションを付けないでコピーすると、リンク元のファイルの中身をファイルとしてコピーしてしまう。`file.symlink-copy`のファイル種別がリンクではなく通常ファイルになっているのがわかる。

```bash
~ ❯❯❯ cp file.symlink file.symlink-copy
~ ❯❯❯ ls -il file.sym*
70679436 lrwxr-xr-x  1 mochiko  2033490572  13  9 19 00:31 file.symlink@ -> file.original
70679879 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:42 file.symlink-copy
70679720 lrwxr-xr-x  1 mochiko  2033490572  10  9 19 00:39 file.symlink2@ -> file.hard2
~ ❯❯❯
```

次の例のふたつは一見同じに見えるが、上段はシンボリックリンクのリンク元である`file.original`をcatしたもので、下段は先ほどの-dなしのシンボリックリンクのコピーによって`file.original`の内容が`file.symlink-copy`のファイルに書き込まれたものをcatしている。リンク元のデータが書き換わっても完全に別のファイルとして存在している`file.symlink-copy`は影響を受けない。

```bash
~ ❯❯❯ cat file.symlink
Hello Linux
~ ❯❯❯ cat file.symlink-copy
Hello Linux
~ ❯❯❯
~ ❯❯❯ echo "Hello World" > file.original
~ ❯❯❯ cat file.symlink
Hello World
~ ❯❯❯ cat file.symlink-copy
Hello Linux
~ ❯❯❯
```

### シンボリックリンクの削除

`rm シンボリックリンク` or `unlink シンボリックリンク`で削除が可能。シンボリックリンクを単純に削除したい場合はunlinkコマンドを利用する方が安全。
rmコマンドを使う際は`rm file.symlink/`とスラッシュを付けて実行するとリンクの参照先のファイル自体が削除されてしまうので注意が必要である。

```bash
~ ❯❯❯ rm file.symlink
~ ❯❯❯ ls -li file.original file.sym*
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.original
70679879 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:42 file.symlink-copy
70679720 lrwxr-xr-x  1 mochiko  2033490572  10  9 19 00:39 file.symlink2@ -> file.hard2
70681303 lrwxr-xr-x  1 mochiko  2033490572  13  9 19 01:00 file.symlink3@ -> file.original
~ ❯❯❯
~ ❯❯❯ unlink file.symlink3
~ ❯❯❯ ls -li file.original file.sym*
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.original
70679879 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:42 file.symlink-copy
70679720 lrwxr-xr-x  1 mochiko  2033490572  10  9 19 00:39 file.symlink2@ -> file.hard2
~ ❯❯❯
```

スラッシュをつけて実行するとリンク元のファイル`file.hard2`自体が削除されている。その結果、`file.symlink2`は参照する先がなくてエラーを生じる。

```bash
~ ❯❯❯ ls -li file.*
70677358 -rw-r--r--  1 mochiko  2033490572   0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.hard2
70677112 -rw-r--r--  2 mochiko  2033490572  12  9 19 00:49 file.original
70679879 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:42 file.symlink-copy
70679720 lrwxr-xr-x  1 mochiko  2033490572  10  9 19 00:39 file.symlink2@ -> file.hard2
~ ❯❯❯
~ ❯❯❯ rm file.symlink2/
~ ❯❯❯ ls -li file.*
70677358 -rw-r--r--  1 mochiko  2033490572   0  9 19 00:11 file.hard-copy
70677112 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:49 file.original
70679879 -rw-r--r--  1 mochiko  2033490572  12  9 19 00:42 file.symlink-copy
70679720 lrwxr-xr-x  1 mochiko  2033490572  10  9 19 00:39 file.symlink2@ -> file.hard2
~ ❯❯❯ cat file.symlink2
cat: file.symlink2: No such file or directory
~ ❯❯❯
```

蛇足。上記のようなまとめを書いていて、途中で「リンク元」と「リンク先」がどっちがどっちか分からなくなった。個人的にオリジナルの方を「リンク元」と呼ぶのに違和感を感じてしまう。参照される先のファイル、というのと混ざるためかもしれない。他の人のブログなどで誤用されていたこともあり、ますますあやふやになって調べたら、同じようなことを書いている人がいて、私だけではなかったのだなと妙な安心を抱いた。

[シンボリックリンクの「リンク元」と「リンク先」](https://farmedgeek.hateblo.jp/entry/20150706/1436152503)

> とにかく「リンク元」が「オリジナル」だって思うことにします。

私もこれで行こうと思います。

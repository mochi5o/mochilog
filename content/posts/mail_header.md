---
title: "メールヘッダーがおもしろい"
date: 2021-04-08T21:15:03+09:00
draft: false
categories:
  - Tech
tags:
  - mail
---

メールヘッダーについて教えてもらったことと調べたこと。

#### メールのヘッダーを雰囲気で読んでいた

仕事柄メールログを見る機会が多く、メールヘッダーを見ながら調査をする機会も割とある。

メールヘッダーについては、

- 上に行くほど受信側に近いので、ヘッダーは下から読むこと
- メール配信の際の経路が記録されていること
- ヘッダーを見てメールソフトがユーザーに送信者などを表示していること

<!--more-->
というようなざっくりした理解だった。
業務の中でヘッダーを読む際に必要なのは配信経路の特定とか、遅延の原因探しとかがほとんどなのと、
ログの調査と並行してチェックするものなので、だいたい上記のようなざっくり理解でしのいでいた。

でも転送設定されたメールのヘッダーを調べたときに混乱したので、質問して教えてもらったことと調べたことをあわせて記しておこうと思う。

メールヘッダーにはさまざまなフィールドがある。
AmazonからGmailに届いた注文確認メールのヘッダーを例としておいておく。

ヘッダーの全体像は、ARCのところとかは途中で切ったけど、それでもけっこう長い。
ヘッダーの長さは経路とかメールサーバーの挙動によって変わるので、もちろんもっと短いのもある。

ARCとかDKIMは別途ちゃんとまとめたいな。

`X-` から始まるヘッダーは拡張フィールド。任意に付与することができる。
逆に言うとそれ以外は決まったフィールドということ。けっこう種類が多い:thinking:

```text
Delivered-To: ******@gmail.com
Received: by 2002:ac2:5293:0:0:0:0:0 with SMTP id q19csp113602lfm;
        Wed, 7 Apr 2021 22:29:41 -0700 (PDT)
X-Google-Smtp-Source: ABdhPJxqmbZpSuBStsM4t7FRkNJT...
X-Received: by 2002:a17:90b:4a81:: with SMTP id ***.154.161785******;
        Wed, 07 Apr 2021 22:29:41 -0700 (PDT)
ARC-Seal: i=1; a=rsa-sha256; t=1617859781; cv=none;
        d=google.com; s=arc-20160816;...
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20160816;....
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@amazon.co.jp header.s=efcs4kiwgez5q6d7nhj6jx2dyn2hqgz4 header.b=HqMXmpdl;
       dkim=pass header.i=@amazonses.com header.s=7v7vs6w47njt4pimodk5mmttbegzsi6n header.b=jt7i+mLC;
       spf=pass (google.com: domain of 202104080***@bounces.amazon.co.jp designates 54.240.25.2 as permitted sender) smtp.mailfrom=202104080***@bounces.amazon.co.jp;
       dmarc=pass (p=QUARANTINE sp=QUARANTINE dis=NONE) header.from=amazon.co.jp
Return-Path: <202104080***@bounces.amazon.co.jp>
Received: from a25-2.smtp-out.us-west-2.amazonses.com (a25-2.smtp-out.us-west-2.amazonses.com. [54.240.25.2])
        by mx.google.com with ESMTPS id *****725pga.357.2021.04.07.22.29.41
        for <******@gmail.com>
        (version=TLS1_2 cipher=ECDHE-ECDSA-AES128-SHA bits=128/128);
        Wed, 07 Apr 2021 22:29:41 -0700 (PDT)
Received-SPF: pass (google.com: domain of 202104080***@bounces.amazon.co.jp designates 54.240.25.2 as permitted sender) client-ip=54.240.25.2;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@amazon.co.jp header.s=efcs4kiwgez5q6d7nhj6jx2dyn2hqgz4 header.b=HqMXmpdl;
       dkim=pass header.i=@amazonses.com header.s=7v7vs6w47njt4pimodk5mmttbegzsi6n header.b=jt7i+mLC;
       spf=pass (google.com: domain of 202104080***@bounces.amazon.co.jp designates 54.240.25.2 as permitted sender) smtp.mailfrom=202104080***@bounces.amazon.co.jp;
       dmarc=pass (p=QUARANTINE sp=QUARANTINE dis=NONE) header.from=amazon.co.jp
DKIM-Signature: v=1; a=rsa-sha256; q=dns/txt; c=relaxed/simple; s=efcs4kiwgez5q6d7nhj6jx2dyn2hqgz4; d=amazon.co.jp; t=1617859781; h=From:Reply-To:To:Message-ID:Subject:MIME-Version:Content-Type:Date; ...
From: "Amazon.co.jp" <digital-no-reply@amazon.co.jp>
Reply-To: digital-no-reply@amazon.co.jp
To: ******@gmail.com
Message-ID: <01010178aff52124-***@us-west-2.amazonses.com>
Subject: Amazon.co.jp ご注文の確認 D01-*****
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary="----=_Part_41084987_160825473.1617859780890"
X-AMAZON-MAIL-RELAY-TYPE: notification
Bounces-to: 202104080***@bounces.amazon.co.jp
X-AMAZON-METADATA: CA=C2GQYSCFOX3CZL-CU=A1ODKHGZ7NLII0
X-Original-MessageID: <urn.rtn.msg.202104080***@1617859780891.rtn-svc-fe-back-c52xlc-81cf1079.us-west-2.amazon.com>
Date: Thu, 8 Apr 2021 05:29:40 +0000
Feedback-ID: 1.us-west-2.rdz**+BU=:AmazonSES
X-SES-Outgoing: 2021.04.08-54.240.25.2
```

送信者に関連するフィールドはこのあたり。
以下、フィールド名に * がついているものは必須のフィールド。

| フィールド名 | 内容 |
| :--- | :---
| From * | 送信者の情報が入るフィールド。
| Reply-to | 返信先として明示的にアドレスを指定したいときに使うフィールド。
| Sender | メールの送信者と作成者が違う場合、送信者の名前やアドレスを記述するフィールド。

メールソフトで返信ボタンを押すと自動的に宛先(=To)に入るメールアドレスは、このヘッダーを見ている。
`Reply-To`があるときはそれが使われるが、必須フィールドではないので`Reply-To`がないときは`From`のフィールドのアドレスが返信に使われる。

宛先に関連するフィールドはこのあたり。
ToとCcはいずれかが必須。

| フィールド名 | 内容 |
| :--- | :---
| To * | メールの宛先を指定するフィールド。CcもしくはToのフィールドで必ず宛先を指定しないといけない。
| Cc * | Carbon Copy
| Bcc | Blind Carbon Copy

おもにメールの管理者向けのヘッダーがこのあたり。

| フィールド名 | 内容 |
| :--- | :---
| Return-path * | 最終的にメールを受信したサーバーが付与する。メール不達の場合などにここにエラーメールが届く。
| Message-ID * | メールを識別するための一意なID。
| Received * | 経路上でメールを受け取ったサーバーが付与する。経路を特定するためのヘッダー。

### 分割してみていく

下が送信元（＝Amazon）なので、下から分割してちょっとずつ見ていく。

ここらへんはAmazon側でつけてるヘッダー。

```text
From: "Amazon.co.jp" <digital-no-reply@amazon.co.jp>
Reply-To: digital-no-reply@amazon.co.jp
To: ******@gmail.com
Message-ID: <01010178aff52124-***@us-west-2.amazonses.com>
Subject: Amazon.co.jp ご注文の確認 D01-*****
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary="----=_Part_41084987_160825473.161"
X-AMAZON-MAIL-RELAY-TYPE: notification
Bounces-to: 202104080***@bounces.amazon.co.jp
X-AMAZON-METADATA: CA=C2GQYSCFOX3CZL-CU=A1ODKHGZ7NLII0
X-Original-MessageID: <urn.rtn.msg.202104080***@1617859780891.rtn-svc-fe-back-c52xlc-81cf1079.us-west-2.amazon.com>
Date: Thu, 8 Apr 2021 05:29:40 +0000
Feedback-ID: 1.us-west-2.rdz**+BU=:AmazonSES
X-SES-Outgoing: 2021.04.08-54.240.25.2
```

送信時のメッセージIDとかSubject、To、Reply-To、Fromなどの情報もここにある。
この内容を解析してメールソフトは件名を表示したり、送信者の情報を表示している。

ヘッダーの内容の一部は送信者側で変更可能なので、実際に送信しているメールアドレスと異なるメールアドレスから送ったように見せたり、受信者自身と異なるメールアドレス宛に送ったものが届いたように見せることが可能である。

よくあるのは送信者が自分で自分宛にとどいた！みたいな。このあたりはスパマーがよく使うやつ。

で、上のほうにいくとGoogleのサーバーがつけるヘッダーがでてくる。一部を抜粋。

```text
Delivered-To: ******@gmail.com
Received: by 2002:ac2:5293:0:0:0:0:0 with SMTP id q19csp113602lfm;
        Wed, 7 Apr 2021 22:29:41 -0700 (PDT)
X-Google-Smtp-Source: ABdhPJxqmbZpSuBStsM4t7FRkNJT...
X-Received: by 2002:a17:90b:4a81:: with SMTP id lp1mr6599080pjb.154.1617859781500;
        Wed, 07 Apr 2021 22:29:41 -0700 (PDT)
Return-Path: <202104080***@bounces.amazon.co.jp>
Received: from a25-2.smtp-out.us-west-2.amazonses.com (a25-2.smtp-out.us-west-2.amazonses.com. [54.240.25.2])
        by mx.google.com with ESMTPS id u30si24670725pga.357.2021.04.07.22.29.41
        for <******@gmail.com>
        (version=TLS1_2 cipher=ECDHE-ECDSA-AES128-SHA bits=128/128);
        Wed, 07 Apr 2021 22:29:41 -0700 (PDT)
Received-SPF: pass (google.com: domain of 202104080***@bounces.amazon.co.jp designates 54.240.25.2 as permitted sender) client-ip=54.240.25.2;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@amazon.co.jp header.s=efcs4kiwgez5q6d7nhj6jx2dyn2hqgz4 header.b=HqMXmpdl;
       dkim=pass header.i=@amazonses.com header....
```

`Received`のフィールドを下から順に追っかけていくと、各サーバーに届いた時間が分かるので、これを見るとメールが遅延した場合などに経路上のどこで時間がかかっていたかが分かる。

一番上の`Delivered-To`が実際に届けられた最終的な宛先になる。

Amazonから送られたメールをGmailのサーバーが受け取ったときの`Recieved`は、今回でいうとこんな感じになってる。見やすいように切り取った。

```text
Received: from us-west-2.amazonses.com           # 送信サーバー
          by mx.google.com                       # 受信サーバー
          with ESMTPS                            # 通信方式
          id *****725pga.357.2021.04.07          # 識別ID
          for <******@gmail.com>                 # 宛先メールアドレス
          Wed, 07 Apr 2021 22:29:41 -0700 (PDT)  # 受信した日時
```

### 問題のReturn-Path

ヘッダーの中で私が勘違いしていたのは、 `Return-Path` というやつ。
名前からしてこれが返信先として使われるフィールドだとばかり思っていたけど、まったくの勘違い。

一般的なメールソフトで返信に使われるのは`Reply-to`もしくは `From`のフィールド。

`Return-Path`には送信者のアドレスがそのまま入っていることが多いが、そうではない場合もある。受信したサーバー（今回だとGmail）が付与するフィールドなので**送信側が付与するものではない**。

このフィールドはエラーが発生したときにメールの作成者にエラーメールを確実に届ける目的で使われる。そのため変更が可能なヘッダーのFromではなく、エンベロープFromの情報を参照する。

メーリングリストなどでは、`Return-Path`にそのメーリングリスト管理者のアドレスが指定されていたり、会社や学校などでシステム管理者がいる場合は組織内の管理者宛に指定されていたりする。

今回だと、`Return-Path: <202104080***@bounces.amazon.co.jp>`となっており、メールはbounces.amaozn.co.jpというバウンスメールを受け取るためのメールアドレスに返すようになっている。

メールめちゃくちゃ奥が深い。そしておもしろい。体系的に学ぶのにいい本とかあるんだろうかー:thinking:

#### 参考にさせていただいたサイト

- https://www.atmarkit.co.jp/fnetwork/rensai/netpro03/mail-header.html
- https://warumono.at.webry.info/201311/article_3.html
- http://blog.smtps.jp/entry/2018/04/18/112026
- http://www.mrfujii.jp/tips/4net/11header.htm

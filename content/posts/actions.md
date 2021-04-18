---
title: "Actionsで特定のラベルがついたissueをスプレッドシートで管理する"
date: 2021-04-18T15:38:24+09:00
draft: false
categories:
  - Tech
thumbnail: "/images/labeled-issue.jpg"
tags:
  - Actions
---

表題の通り。
会社で問合せ内容をissue管理しているのだが、それをデータベースにして傾向分析をしたい、となった。

スプレッドシートにissueの一覧を持ってきて、問合せの調査結果などを踏まえて事象をうまく分類していきたい。
最終的にはDatastudioにインポートしていい感じに傾向の可視化ができれば、というようなモチベーション。

GitHubのAPIを使って直接Datastudioにインポートすることもできるし、そういうデータ基盤づくりも進んではいる。
がしかし、対応結果などをそれに紐付けて分類していくとなると、より自由度の高いスプレッドシート上で管理・編集するのが都合が良かった。
それとGithubのAPIでまとめて取得するよりもできるだけリアルタイムでスプレッドシートに反映させたかったというのがありこのような仕組みを考えた。

## ActionsとGASの準備

### issueで特定のラベルが付いたらアクションを実行

これは調べたら割とすぐ出てきた。
まず、issueにラベルが付与されたときに実行する方法は、こんなかんじ。

```yml
on:
  issues:
    types:
      ['labeled']
```

これでラベルが付与されたときに実行されるアクションがつくれる。
特定のラベルがついたときだけに限定したい場合は、つづけて条件を記載することもできる。

```yaml
if: |
  (github.event.label.name == 'wontfix') || (github.event.label.name == 'bug')
```

### 日付の取得

アクションが実行される日時をこんなふうに取得できる。
Tokyoにタイムゾーンを設定しておけばOK。
これを使ってラベルがついた日時をスプレッドシート側に持っておきたかった。
当初、ラベルのつけ外しで時間計測もできるかな？という目論見もあり、こちらに実装しているが、
GAS側でPOSTされた日時を得ることも可能だし、今回の目的に限れば別にこの方法でなくてもどうとでもなりそう。

```yaml
- name: Setup timezone
  uses: zcong1993/setup-timezone@master
  with:
    timezone: Asia/Tokyo
- name: Get current date-time
  id: date
  run: echo "::set-output name=date::$(date +'%Y%m%d-%H%M')"
```

### GASでスプレッドシート書き込み用のエンドポイントを用意する

これも調べたら割とすぐ出てくる。
doPostという関数が用意されてるので、そこにデータを突っ込むだけ。
スプレッドシートからスクリプトエディタを起動して、編集→保存→デプロイする。
すべてのユーザーに公開しないとcurlからたたけないので注意。

IDEの自動保存に慣れてしまっている私は、保存を忘れて変更が反映されてないままデプロイする、というのを数回繰り返してしまった。

```javascript
function doPost(e) {
  var ss       = SpreadsheetApp.getActiveSpreadsheet();
  var sheet    = ss.getSheetByName('issue data'); 
  var PostData = JSON.parse(e.postData.contents);
  insertPostData(sheet, PostData);
}

function insertPostData(sheet, data) {
  var url = data.url;
  var title = data.title;
  var label = data.label;
  var time = data.time;
  sheet.appendRow([url, title, label, time]);
}

```

## ActionsからデータをPOST

envをつかってアクションの中で使う環境変数を定義できる。
今回テストのためにissue上にもコメントを残して、GASにもPOSTする、というのをやったのでこうなっている。
GitHubのissueにコメントをPOSTするにはアクセストークンが必要なので別途用意して、リポジトリのsecretsに設定している。
`${{ secrets.ACCESS_TOKEN }}` という形でsecretsの値は取得できる。

おなじく先程デプロイしたURLもsecretsに保存しておいて、そこに対してデータをPOSTする。

```yaml
- name: Data post
  env:
    GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }}
    URL: ${{ github.event.issue.url }}
    LABEL_NAME: ${{ github.event.label.name }}
    TIME: ${{ steps.date.outputs.date }}
    TITLE: ${{ github.event.issue.title }}
  run: |
    comments="$GITHUB_API_URL"
    curl -X POST -H "Authorization: token $GITHUB_TOKEN" -d "{\"body\": \"$TIME\n$LABEL_NAME\"}" ${{ github.event.issue.comments_url }}
    curl -X POST -H "Content-Type: application/json" -d "{\"url\": \"$URL\", \"title\": \"$TITLE\", \"label\": \"$LABEL_NAME\", \"time\": \"$TIME\"}" -L ${{ secrets.SPREAD_SHEET }}

```

## 実行結果

こういうかんじで、ラベルをつけるとすぐにActionsがはしる。
ラベルの下のコメントはActionsからPOSTされたもの。

![labeled issue](https://user-images.githubusercontent.com/41158022/115138717-be9dfd00-a068-11eb-87d6-aff424efd113.png)

Actionsタブで実行の様子が確認できる。
![Actions](https://user-images.githubusercontent.com/41158022/115138721-cb225580-a068-11eb-9640-a3f345b133ef.png)

スプレッドシートに追加された。
こちらで更に詳細な分類を入力して、Datastudioとかに突っ込んで分析に役立てることができる。
![spread sheet](https://user-images.githubusercontent.com/41158022/115138736-d9707180-a068-11eb-96ba-743ec4b9f6ca.png)

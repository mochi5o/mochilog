---
title: "Notion APIパブリックベータ版をつかう"
date: 2021-05-22T16:06:11+09:00
draft: false
categories:
  - Tech
tags:
  - Notion
---

Notion APIをさわってみた。
前回、GitHub Acitonsを使って、特定のラベルがついたissueをスプレッドシートに自動でPOSTする処理を書いたので、それをアレンジして投稿先をNotionにしようと思った。

## Notion APIのつかいかた

<!--more-->

とにかく、かんたんに使い始めることができる。

[Getting Started](https://developers.notion.com/docs/getting-started)のページを参照すると、curlをつかった一番シンプルな形のPOSTはすぐにできる。
以下、試しにcurlでいろいろなパラメータをPOSTしたときの手順。

### Integrationを登録する

Notionのアカウントは持っている前提。また基本的なページ作成、データベース作成については言及しない。

左メニューのSetting > Integrations > Develop your own Integrations と選択すると、Integrationの新規設定画面が表示される。
必要事項を入力するだけで新規Integrationが登録できる。

登録できたら、一覧画面に表示される。
{{< figure src="/images/my_integration.jpg" class="center" width="80%">}}

詳細画面ではTOKENが確認できる。権限設定もここで変更可能。
{{< figure src="/images/integrations_informations.jpg" class="center" width="80%">}}

## データベースにIntegrationをひもづける

自分のワークスペースでAPIにひもづけたいデータベースを作成して、左上のShareメニューから先程作成したIntegrationを選択してひもづける。

{{< figure src="/images/notion_menu.jpg" class="center" width="80%">}}

これで、POSTする準備が整った。

テスト用のコマンドは[Getting Started](https://developers.notion.com/docs/getting-started)のページにのってるので、それをそのまま実行してみるとなんなく成功する。

```bash
curl -X POST https://api.notion.com/v1/pages \
  -H "Authorization: Bearer {MY_NOTION_TOKEN}" \
  -H "Content-Type: application/json" \
  -H "Notion-Version: 2021-05-13" \
  --data '{
    "parent": { "database_id": "{DATABASE_ID}" },
    "properties": {
      "Name": {
        "title": [
          {
            "text": {
              "content": "Yurts in Big Sur, California"
            }
          }
        ]
      }
    }
  }'
```

これで最初のテストデータの投稿が完了した。

## もう少しいろいろなパラメータをPOSTする

Notionのデータベースでは、いわゆるレコードにあたる各行はそれぞれ独立したページとして存在する。データベースの状態でみたときに、カラムに相当するものは、各ページのpropatyと呼ばれるオブジェクトとして表現されている。

先程登録したパラメータはtitleという特殊なpropatyで、そのままページのタイトルになる部分。このあたりの説明は[こちら](https://developers.notion.com/docs/working-with-databases)に詳しい。

今度は、他のpropaty（カラム）にもデータを登録できるようにしたい。[Postmanのワークスペース](https://www.postman.com/notionhq/workspace/notion-s-public-api-workspace/overview)も公開されているので参考にしながらすすめる。

とりあえずNumber、Text、Date、Selectのpropatyに対して投稿できることを確認した。

{{< figure src="/images/post_for_notion.jpg" class="center" width="80%">}}

前回作ったActionsから投稿してみる。

以下がActionsのYAMLファイル。
curlで叩くためにヒアドキュメントにしたけど、これだと変数展開されないので、複雑なJSONをPOSTするならjsで書いたほうが良い。

```yml
name: labels-actions
 on:
   issues:
     types:
       ['labeled', 'unlabeled']

 jobs:
   labeled-actions:
     runs-on: ubuntu-latest
     if: |
       (github.event.label.name == '対応中') || (github.event.label.name == '依頼中')
     name: Recording label's add or remove time
     steps:
       - name: Checkout
         uses: actions/checkout@v2
       - name: POST Comment
         env:
           GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }}
           LABEL_NAME: ${{ github.event.label.name }}
           TITLE: ${{ github.event.issue.title }}
         run: |
           curl -X POST -H "Authorization: token $GITHUB_TOKEN" -d "{\"body\": \"ラベルが付きました\nラベル：　$LABEL_NAME\nタイトル：　$TITLE\"}" ${{ github.event.issue.comments_url }}
           data=$(cat <<EOF
           {
             "parent": { "database_id": "dfc2cdc260ae476c8f2f5d4065f9a94a" },
             "properties": {
               "Name": {
                 "type": "title",
                 "title": [{ "text": { "content": "POST Data" } }]
               },
               "Tags": {
                 "type": "rich_text",
                 "rich_text": [{ "text": { "content": "actions" } }]
               },
               "Price": {
                 "type": "number",
                 "number": 500
               },
               "Date": {
                 "type": "date",
                 "date": { "start": "2021-05-11" }
               },
               "Status": {
                 "select": {
                   "name": "complete"
                 }
               }
             }
           }
           EOF
           )
           curl -X POST "https://api.notion.com/v1/pages" \
             -H "Authorization: Bearer ${{ secrets.MY_NOTION_TOKEN }}" \
             -H "Content-type: application/json" \
             -H "Notion-Version: 2021-05-13" \
             -d "$data"
```

上記を実行すると、GitHubの該当issue上にもコメントが追加される。

{{< figure src="/images/github_comment.jpg" class="center" width="80%">}}

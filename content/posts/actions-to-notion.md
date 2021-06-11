---
title: "特定のラベルをつけたissueをNotion上でタスク管理する"
date: 2021-06-01T01:37:41+09:00
draft: false
categories:
  - Tech
tags:
  - Notion
  - GitHub Actions

---

最近触っているGitHub ActionsとNotion API。GitHub上で特定のラベルがついたissueをNotion側にPOSTしてNotionでタスク管理できるようにした。

Notionに寄せたいモチベーションは、リポジトリが別でもNotion上でタスクとして一元管理できる点と、プロパティが自由に追加できるのでステータスとか分類の管理が柔軟な点、それとフィルター機能が充実していること。

Notionが公式提供しているSDKをつかって実装することにした。

- Notion SDK: [https://github.com/makenotion/notion-sdk-js](https://github.com/makenotion/notion-sdk-js)
- API documentation: [https://developers.notion.com/reference](https://developers.notion.com/reference)
- Postman collection: [https://www.postman.com/notionhq/](https://github.com/makenotion/notion-sdk-js)

## ActionsからNotionのAPIを使ってissueのデータをPOSTする

<!--more-->

ということで早速実装。

issueに特定のラベルがつくと走るActionsを作成する。
こんな感じで書いておけば、issueに対してlabeledイベントが発生したときに、そのラベルを確認して条件に合致すれば以降のステップに進むように制御できる。

```main.yml
on:
  issues:
    types:
      ['labeled']
jobs:
  labeled-actions:
    runs-on: ubuntu-latest
    if: |
      contains(github.event.label.name, '特別なラベル')
    steps:
...
```

特定のstepを実行するときにリポジトリに定義したsecretsやActionsのイベントの情報を使いたいときは、yml上で`with`を使って定義してあげれば渡すことができる。
これを使って、Actionsが発火したissueの情報やsecretsに設定したNotionのIntegration Tokenを渡しておく。

```main.yml
with:
  issue-title: ${{ github.event.issue.title }}
  url: ${{ github.event.issue.url }}
  integrations-token: ${{ secrets.NOTION_TOKEN }}
  db-id: ${{ secrets.DB_ID }}
```

ちなみに、あらかじめNotion側で任意のデータベースのページでIntegrationの設定を入れておく必要があるので注意。

[Share a database with your integration](https://developers.notion.com/docs/getting-started#step-2-share-a-database-with-your-integration)

必要な情報を渡すことができれば、あとはそれをもとにNotion SDKを使ってNotionにPOSTする処理を書けばよい。

- Notion SDK: [https://github.com/makenotion/notion-sdk-js](https://github.com/makenotion/notion-sdk-js)

npmでインストールするだけで使える。

```bash
npm install @notionhq/client
```

以下は記述方法の抜粋。

```main.js
const { Client, LogLevel } = require('@notionhq/client');

const main = async () => {
  const notionToken = core.getInput('integrations-token');
  const issueTitle = core.getInput('issue-title');
  const url = core.getInput('url');
  const dbId = core.getInput('db-id');

  const notion = new Client({
    auth: notionToken,
    logLevel: LogLevel.DEBUG,
  });
  const parent = {
    database_id: dbId,
  };
  const properties = {
    Title: {
      title: [{ text: { content: issueTitle } }],
    },
    Url: {
      url: url,
    },
    Status: {
      select: {
        name: 'not started',
      },
    },
  };
  const response = await notion.pages.create({
    parent: parent,
    properties: properties,
  });
...
```

実装はこれだけで、非常にシンプルに使えた:smile:
特定のラベルをつけるとこんな風にNotionにテーブルが追加される。やったー！

今回はNotion側にformulaを設定して、Notion上でのページのcreated_atの時間と、Notionのページの最終更新までの時間の差を分で算出するようにしてみた。
これでタスク終わりにcompleteをつけてあげるだけで、issueの登録から解決までの時間を簡易的に計測できる。Notionのformulaはちょっとしたことに便利。

{{< figure src="/images/issues.jpg" class="center" width="80%">}}


### ActionsのTips

本題からはずれるが、jsのAction実行時はnodeの環境を使い、リポジトリをチェックアウトしてそのコードを利用するので`node_modules`の依存ライブラリも同梱する必要がある。でも`node_modules`を一緒にリポジトリにアップするのはうっとおしい。で、どうするかというと、nccというツールを使うことを知ったのでメモしておく。

Actionsのドキュメントに書いてあった。

- [アクションの GitHub へのコミットとタグ、プッシュ](https://docs.github.com/ja/actions/creating-actions/creating-a-javascript-action#commit-tag-and-push-your-action-to-github)
- [@vercel/ncc](https://github.com/vercel/ncc)

nccを使って、

```bash
ncc build index.js
```

とすれば、依存関係をコンパイルした `dist/index.js`を生成してくれる。これをリポジトリにプッシュして、Actions実行時にはこのファイルを実行するようにすればよい。

これで`node_mopdules`ディレクトリ以下をgitignoreできる。
なるほどー。

### リポジトリ

- [mochi5o/from_actions_to_notion](https://github.com/mochi5o/from_actions_to_notion)

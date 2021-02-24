---
title: "Hugoのテーマのバグを踏んだ"
date: 2021-02-24T23:22:42+09:00
tags: [
    "hugo",
    "oss",
]
categories: [
    "日々のこと",
]
---

めちゃくちゃ軽微なバグを踏んだので、いい機会だと思ってドキドキしながら初のバグレポートをした話。
軽微すぎるんですが、シュッと直せるほどの力も余裕もなかったので報告だけです。

<!--more-->

### 選択したテーマ

clarityというテーマを選択。

[https://themes.gohugo.io/theme/hugo-clarity](https://themes.gohugo.io/theme/hugo-clarity/)

hugoは手元でサーバー立ち上げて確認しながら書けるし、mdで書くので基本的にすごく簡単。
だったはずが、ひと通りメニューとか整理しおわって「だいたいいいかなー」ってなったタイミングで、あることに気づ。

なんか、ブログ本文の最後に**true**って謎の文字列？！

テーマ？それともhugoの設定？というのがパッと分からず「もしかしたらローカル環境でだけ表示される変数なのか…」とかいろいろ考えを巡らせたが、結局テーマ側のバグっぽい。

どうやら[この辺り](https://github.com/chipzoller/hugo-clarity/blob/master/layouts/_default/single.html#L26-L36)のコードがtrueという文字列を出力してしまってるようで、コメント欄を使う予定もないので、全部バッサリ削除することに。

テーマのカスタマイズ方法は`themas/hugo-clarity/layouts`の下にあるファイルを、`layouts`ディレクトリで上書きするということだったので、`layouts/_default/single.html`を新規作成して、不要な箇所を削除してファイルを設置すると、きれいに**true**が消えてくれた。

ちなみに、2/23時点では[テーマのDemoサイト](https://themes.gohugo.io/theme/hugo-clarity/)でも同様の文字列が出現しているのだけど、同じバグに悩んでる人がいなかったみたいなので直近のコミットで入ったバグをたまたま踏んだのかも…

ちょうど[3日前のこのコミット](https://github.com/chipzoller/hugo-clarity/commit/f368af7333b1eece63925aa0a08bae553e980211#diff-b2ff2a8d1f2a8a15c65288757c8b9acde7bec0ef491acb8048f1ddb61985cc41)だった！

ということで[バグレポートのissue](https://github.com/chipzoller/hugo-clarity/issues/145)を投げてみました。

OSSにissue投げるの初めてで緊張。
コメントもらえるかなぁ…と思って朝起きたらすぐ修正してくれてました:raised_hands:

---
title: "GitHub の PR で編集されていない箇所にレビューコメントしたかった"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Git", "GitHub", "コードレビュー"]
published: true
---

## 概要

GitHub の PR で編集されていない箇所にレビューコメントしたかったのですが、どうしてもできなかったため代替策を記録しておきます。

## 課題

GitHub で コードレビューをしていると、その PR 内で編集されていない箇所にコメントしたいことがあります。ですが PR では編集された近辺の行にしかコメントを追加できません。

![alt](/images/github-pr-comment-20240424/1.png)
*コード編集箇所とその近辺では、青い＋マークが表示される*

![alt](/images/github-pr-comment-20240424/2.png)
*＋マークをクリックし、コメントを追加する*

![alt](/images/github-pr-comment-20240424/3.png)
*しかし編集箇所ではないところには＋マークが表示されずコメントできない*

## 対策

### 基本

この場合、PR のコード内に直接コメントをできる方法はありません。そのため以下の代替手順でコメントを残すことにしました。

1. View File を選択してファイルを開く
1. 該当する行を選択して Copy permalink をクリック
1. Conversation にコピーした URL を含めてコメント
1. Request changes にコメントを書いて Submit review をクリック

![alt](/images/github-pr-comment-20240424/4.png =400x)
*View File を選択してファイルを開く*

![alt](/images/github-pr-comment-20240424/5.png =400x)
*該当する行を選択して Copy permalink をクリック*

:::message
複数行を選択するには、先頭行をクリック→末尾行をShift+クリックしてください
:::

![alt](/images/github-pr-comment-20240424/6.png)
*Conversation にコピーした URL を含めてコメント*

![alt](/images/github-pr-comment-20240424/7.png)
*Conversation にコピーした URL を含めてコメント*

![alt](/images/github-pr-comment-20240424/8.png)
*Request changes にコメントを書いて Submit review をクリック（自作自演 PR のため Request changes にチェックが入れられず）*

### レンダリングされるファイルの場合

Markdown や CSV などレンダリング表示されるファイルの場合、Copy permalink が選択できない場合があります。この場合はコード表示に切り替えることで同様の操作が可能です。

![alt](/images/github-pr-comment-20240424/9.png)
*Markdown では Copy permalink などのメニューが表示されない*

![alt](/images/github-pr-comment-20240424/10.png)
*Code ボタンを押す事でコード表示に*

## まとめ

どの箇所をどのように直すべきか一目でわかるようにしてあげる事で、レビュアー/レビュイー間の齟齬がなくなり結果的に早く仕事が進みます。良いコードレビューを！

---
title: "GitHub の Saved Replies でコードレビューに心理的安全性を！"
emoji: "🐈‍⬛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Git", "GitHub", "コードレビュー"]
published: true
---

## はじめに

コードレビューをする際に、「個人の好みかなぁ」というコメントを付けるのをためらったことはありませんか？レビュイーに正しく意図を伝えて心理的安全性を高めたいと思い、Saved Replies を使ってコメント種別をつけることにしました。

## コメント種別

|略語|意味|説明|
|----|----|----|
|MUST|必須：Must|修正されないとApproveできません。修正のアクションが必要です。|
|IMO|提案：In My Opinion|個人的な見解や軽微な提案です。タスク化や修正等のアクションが必要です。|
|Q|質問：Question|質問です。回答のアクションが必要です。|
|NR|お手すきで：No Rush|不急なので今やるかはおまかせします。タスク化や修正等のアクションが必要です。|
|NITS|あらさがし：Nitpick|重箱の隅をつつく提案です。やってもいいがやらなくてもよいです。アクションは不要です。|
|FYI|参考までに：For Your Information|参考までに共有します。アクションは不要です。|

## Saved Replies

返信テンプレートともいいます。Saved Replies を活用することで、手間なくコメントをつけることができます。

### 設定方法

1. <https://github.com/settings/replies> へアクセス
1. タイトルとコメントを記載
1. Add saved reply をクリックし保存

![alt](/images/github-saved-replies-20240426/1.png =600x)

### 返信テンプレート例

```md
**[MUST: 必須]**[^1]

xxx

[^1]: 修正されないとApproveできません。修正のアクションが必要です。
```

```md
**[IMO: 提案]**[^1]

xxx

[^1]: 個人的な見解や軽微な提案です。タスク化や修正等のアクションが必要です。
```

```md
**[Q: 質問]**[^1]

xxx

[^1]: 質問です。回答のアクションが必要です。
```

```md
**[NR: お手すきで]**[^1]

xxx

[^1]: 不急なので今やるかはおまかせします。タスク化や修正等のアクションが必要です。
```

```md
**[NITS: あらさがし]**[^1]

xxx

[^1]: 重箱の隅をつつく提案です。やってもいいがやらなくてもよいです。アクションは不要です。
```

```md
**[FYI: 参考までに]**[^1]

xxx

[^1]: 参考までに共有します。アクションは不要です。
```

### 使用例

Saved replies をクリックすることで、返信が選択できます。タイトルで検索も可能です。

![alt](/images/github-saved-replies-20240426/2.png =250x)
![alt](/images/github-saved-replies-20240426/3.png =500x)

実際にコメントすると、以下のような感じになります。

![alt](/images/github-saved-replies-20240426/4.png)
![alt](/images/github-saved-replies-20240426/5.png)
![alt](/images/github-saved-replies-20240426/6.png)
![alt](/images/github-saved-replies-20240426/7.png)
![alt](/images/github-saved-replies-20240426/8.png)
![alt](/images/github-saved-replies-20240426/9.png)

## まとめ

レビュアーとしては軽い提案のつもりでも、レビュイーには厳しい指摘コメントばかりに見えることがあります。Saved Replies を駆使して、意図を正確に伝えられると快適なコードレビューになると思います！

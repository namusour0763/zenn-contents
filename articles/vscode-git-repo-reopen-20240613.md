---
title: "VSCode で Git リポジトリが表示されない場合の対処"
emoji: "🔌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["VSCode", "Git"]
published: true
---

## リポジトリが表示されない

VSCode で Git 管理されているフォルダやファイルを開いているのに、ソース管理タブでリポジトリが表示されず時間を使ってしまったのでメモを残します。

![alt](/images/vscode-git-repo-reopen-20240613/workspace.png =400x)
*`zenn-contents` フォルダ内のファイルを開いているが...*

![alt](/images/vscode-git-repo-reopen-20240613/git.png =400x)
*ソース管理に表示されずコミット等操作ができない*

## 対処方法

`Ctrl + Shift + P` でコマンドパレットを呼び出し、`Reopen Closed Repositories` を選択すれば OK です。

![alt](/images/vscode-git-repo-reopen-20240613/reopen1.png =550x)
*Reopen Closed Repositories を選択*

![alt](/images/vscode-git-repo-reopen-20240613/reopen2.png =550x)
*表示するリポジトリを選択*

## コメント

VSCode が Git のリポジトリを認識しないと思っていましたが、どうやら邪魔になったとき「リポジトリを閉じる」をクリックしてしまったようです。

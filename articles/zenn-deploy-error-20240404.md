---
title: "Zenn へのデプロイが失敗した場合の対応"
emoji: "⚠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Zenn", "GitHub"]
published: true
---

## 結論

GitHub リポジトリの該当ファイルに何らかの変更を加えて再プッシュすれば OK。

## デプロイ失敗

GitHub 上で管理している記事へ軽微な変更を加えプッシュしたところ、デプロイが終了しませんでした（通常数秒で完了）。

1 時間経ったところでタイムアウトのエラーを吐いて停止。デプロイ停止やリトライなどのボタンは存在しませんでした。

![image](/images/zenn-deploy-error-20240404/1.png)
![image](/images/zenn-deploy-error-20240404/2.png)

## エラー内容

> タイムアウトしました。一度に多数のファイルは更新できないため、複数回のプッシュに分けてください。

とのことですが `1 changed file with 7 additions and 5 deletions.` のため、問題のあるサイズとは考えられません。

> 更新されたファイルはありません

という表示とも矛盾しており、Zenn もしくは GitHub に起因する内部的なエラーと思われます。

## 対応

軽微な変更を加えて再プッシュすると通常通りデプロイに成功。変更を打ち消すコミットを更にプッシュすることで、理想の状態に戻りました。

![image](/images/zenn-deploy-error-20240404/3.png)
![image](/images/zenn-deploy-error-20240404/4.png)
![image](/images/zenn-deploy-error-20240404/5.png)

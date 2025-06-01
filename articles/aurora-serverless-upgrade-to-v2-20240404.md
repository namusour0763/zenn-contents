---
title: "Aurora Serverless v1 から v2 への移行手順"
emoji: "🗄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "RDS", "Aurora"]
published: true
---

## はじめに

2024 年 12 月 31 日に Aurora Serverless v1 はサポートを終了します。ここでは Aurora Serverless v2 へアップグレードする手順を紹介します。

### Aurora Serverless v1 と v2 の違い

Aurora Serverless v1 と v2 は DB クラスターおよびインスタンスに大きな違いがあるので、事前に概念を確認してください。

- DB クラスター
  - v1 は Serverless
  - v2 は Provisioned
- DB インスタンス
  - v1 は インスタンスが存在しない
  - v2 は Serverless（Provisioned インスタンスと共存可）

以下記事の「Provisioned と Serverless の違い」が参考になります。

https://dev.classmethod.jp/articles/moving-from-aurora-serverless-v1-tov2/

### アップグレード前提

- Aurora Serverless v2 が対応するエンジンバージョン (Aurora PostgreSQL 13.12) を利用
- アップグレードにブルー/グリーンデプロイは利用しない

対応するエンジンバージョンおよびエンジンバージョンのアップグレード方法は以下を参照してください。

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.ServerlessV2.html#Concepts.Aurora_Fea_Regions_DB-eng.Feature.ServerlessV2.apg

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.modifying.html#aurora-serverless.modifying.upgrade

## アップグレードパス

アップグレード手順の概要です。公式ドキュメントも確認してください。

1. Aurora Serverless v1 DB クラスターをプロビジョニングされた DB クラスターに変換
1. Aurora Serverless v2 リーダー DB インスタンスを DB クラスターに追加
1. Aurora Serverless v2 DB インスタンスにフェイルオーバー
1. リーダー DB インスタンスの追加（オプション）
1. プロビジョニングされた DB インスタンスの削除（オプション）

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.upgrade.html#aurora-serverless-v2.upgrade-from-serverless-v1-procedure

## アップグレード手順

### Aurora Serverless v1 クラスターの用意

説明用に Severless v1 を用意します。AWS CLI を実行します。

:::message
2024 年 4 月現在、マネジメントコンソールから Aurora Serverless v1 を起動することはできません。
:::

:::message
アクセスキー発行はせず、CloudShell, Cloud9, IAM ロールアタッチ済みの EC2 での実行をおすすめします。
:::

```bash
aws rds create-db-cluster --db-cluster-identifier sample-cluster \
    --engine aurora-postgresql --engine-version 13.12 \
    --engine-mode serverless \
    --scaling-configuration MinCapacity=2,MaxCapacity=4,SecondsUntilAutoPause=1000,AutoPause=true \
    --master-username username master-user-password password \
    --db-subnet-group-name sample-db-subnet-group
```

![image](/images/aurora-serverless-upgrade-to-v2-20240404/1.png)

### プロビジョニングされた DB クラスターへ変換

AWS CLI を実行してクラスターを変換します。

:::message
この操作はマネジメントコンソールからでは行えません。
:::

:::message
Unknown options: エラーが出た場合、新しい AWS CLI に更新してください。
:::

```bash
aws rds modify-db-cluster \
    --db-cluster-identifier sample-cluster \
    --engine-mode provisioned \
    --allow-engine-mode-change \
    --db-cluster-instance-class db.t3.medium
```

![image](/images/aurora-serverless-upgrade-to-v2-20240404/2.png)

### Aurora Serverless v2 リーダー DB インスタンスの追加

マネジメントコンソールから Serverless v2 のインスタンスを追加します。

![image](/images/aurora-serverless-upgrade-to-v2-20240404/3.png)
![image](/images/aurora-serverless-upgrade-to-v2-20240404/4.png)
![image](/images/aurora-serverless-upgrade-to-v2-20240404/5.png)

### Serverless v2 リーダー DB インスタンスへフェイルオーバー

マネジメントコンソールからフェイルオーバーを実行します。

![image](/images/aurora-serverless-upgrade-to-v2-20240404/6.png)
![image](/images/aurora-serverless-upgrade-to-v2-20240404/7.png)

### リーダー DB インスタンスの追加

可用性確保のため、別の AZ にリーダーを追加します。

![image](/images/aurora-serverless-upgrade-to-v2-20240404/8.png)
![image](/images/aurora-serverless-upgrade-to-v2-20240404/9.png)
![image](/images/aurora-serverless-upgrade-to-v2-20240404/10.png)

### プロビジョニングされた DB インスタンスの削除

不要になったプロビジョニングされた DB インスタンスを削除します。

![image](/images/aurora-serverless-upgrade-to-v2-20240404/11.png)
![image](/images/aurora-serverless-upgrade-to-v2-20240404/12.png)

## 移行手順まとめ

キーポイントは以下の通りです。

- Aurora Serverless v2 への移行は、複数のステップを踏む必要がある
- アップグレード作業はマネジメントコンソールでは完結せず、AWS CLI の利用が必要

### その他耳より情報

- Serverless v1 の DB クラスタースナップショットからは、マネジメントコンソールのみで Serverless v1 のリストアができることを確認
- 許容できるのであれば、スナップショットからプロビジョニングされたクラスターと Serverless v2 インスタンスとしてリストアする方法も取れる

---
title: "CloudShell を VPC で使ってみた！"
emoji: "🐚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "VPC"]
published: true
---

## はじめに

2024/06/26 に CloudShell が VPC をサポートするようになったので使ってみます！

https://aws.amazon.com/jp/about-aws/whats-new/2024/06/aws-cloudshell-amazon-virtual-private-cloud/

## VPC サポートのメリット

VPC をサポートするということは、より CloudShell をセキュアに利用できるということです！具体的には以下のネットワーク制御が効くようになります。

- ルートテーブル
- セキュリティグループ
- ネットワーク ACL
- AWS Network Firewall
- その他 ネットワークセキュリティ製品

## 起動方法

CloudShell を開き、右上の「アクション」から「Create VPC environment (max 2)」をクリック。名前・VPC・サブネット・セキュリティグループを入力し「Create」から起動します。

![image](/images/cloudshell-in-vpc-20240702/create_vpc_env.png)
![image](/images/cloudshell-in-vpc-20240702/create_vpc_env2.png =500x)

本当に VPC 内で起動できているのでしょうか？ENI を確認したところ、指定したサブネットで起動していることがわかりました！

![image](/images/cloudshell-in-vpc-20240702/details.png =500x)
![image](/images/cloudshell-in-vpc-20240702/eni1.png)
![image](/images/cloudshell-in-vpc-20240702/eni2.png)

## 運用上の制約

パブリックサブネットで動作確認をしたところ、AWS の API コールもインターネットへの接続もできません。

![image](/images/cloudshell-in-vpc-20240702/public_subnet.png)

公式ページを読むと以下の制約を発見。

```text
AWS CloudShell 環境は、プライベート VPC サブネット内にある場合にのみインターネットに接続できます。
```

:::message
CloudShell VPC 環境には、パブリック IP アドレスは割り当てられない、ということ。
:::

https://docs.aws.amazon.com/cloudshell/latest/userguide/using-cshell-in-vpc.html

詳細はリンク先を確認ください。他にも制約があるので、簡単にまとめておきます。

- VPC 環境は最大 2 つ
- セキュリティグループは最大 5 つ
- アップロード・ダウンロードアクションは使用不可
- ストレージは一時的、セッションを終了するとデータは削除
- インターネットアクセスはプライベートサブネットでのみ使用可
- API コールには AWS サービスへのネットワークアクセス必須

## 動作確認

プライベートサブネットでしか使えないことがわかったので、インターネットアクセスを確保するため NAT ゲートウェイを作成し、ルートテーブルのルートを変更します。

![image](/images/cloudshell-in-vpc-20240702/vpc.png)

このプライベートサブネットで CloudShell を起動すると、**AWS の API コール、インターネットアクセス、EC2 への Ping ができる** ことを確認できました！

![image](/images/cloudshell-in-vpc-20240702/private_subnet.png)
![image](/images/cloudshell-in-vpc-20240702/ping_instance.png)

VPC で起動できるということは、**VPC で稼働する RDS にもアクセスできるのでは？** ということで確認します。CloudShell には `psql` コマンドが入っているため、難なく接続できました！

![image](/images/cloudshell-in-vpc-20240702/connect_rds.png)

Aurora であればマネジメントコンソールからクエリエディタで SQL を発行できますが、それ以外では踏み台等からのアクセスが必要です。CloudShell を使うと楽できる場面があるかもしれません。

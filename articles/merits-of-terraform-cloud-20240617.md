---
title: "HCP Terraform/Terraform Cloudのメリットをざっくりご紹介！"
emoji: "✈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Terraform", "CICD", "AWS"]
published: true
---

## はじめに

HCP Terraform が気になる・使ってみたいという人向けにメリットをざっくりご紹介します！

:::message
2024/4/22 に Terraform Cloud は HCP Terraform と名称変更されました。
:::

https://developer.hashicorp.com/terraform/cloud-docs

## HCP Terraform のメリット

### どこからでも実行可能

`terraform login` コマンドでログインすれば、どんな端末でもすぐに同じ環境を使えます。複数端末あっても楽ちんです。

![alt](/images/merits-of-terraform-cloud-20240617/login.png =600x)

### 実行環境がクラウド

クラウドで実行されるので、サーバーの面倒を見る必要がありません。メンバー端末の環境差分なども関係なくなります。ステートファイルのロックや保管も気にしなくて良いですね。

![alt](/images/merits-of-terraform-cloud-20240617/plan.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/run.png =600x)

### アクセスキーが不要

各種クラウドサービスと OpenID Connect (OIDC) で連携でき、非常にセキュアです。アクセスキーの発行は不要です。~~アクセスキーを使う記事が多すぎるので滅ぼしたい。~~

![alt](/images/merits-of-terraform-cloud-20240617/id_provider.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/variables.png =600x)

### CI 実装が簡単

画面をポチポチすればすぐ CI/CD が実装できます。CI の整備やメンテナンスも馬鹿にならないので助かります。

![alt](/images/merits-of-terraform-cloud-20240617/ci_1.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/ci_2.png =600x)

### GitHub 連携がイケてる

プルリクエストの画面から Plan 結果や Apply 結果がいい感じに確認できます。レビュアーも大助かりです。

![alt](/images/merits-of-terraform-cloud-20240617/pr.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/checks1.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/checks2.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/checks3.png =600x)

### ガバナンスを効かせやすい

権限を持った人による承認がないと Apply できない設定ができます。コードがマージされたら自動で Apply もできますが、ガバナンスを効かせたい・タイミングを調整したい場合に。

![alt](/images/merits-of-terraform-cloud-20240617/apply1.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/apply2.png =600x)
![alt](/images/merits-of-terraform-cloud-20240617/apply3.png =600x)

## まとめ

メリットが多いので、ぜひ一度体験してみてください！

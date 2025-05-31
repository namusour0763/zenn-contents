---
title: "AWS Systems Manager ドキュメント (SSM ドキュメント) で EC2 をプロビジョニング "
emoji: "⚙️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "SystemsManager", "EC2", "Ansible"]
published: true
---

## まえがき

本記事では SSM ドキュメントを利用して、EC2 をプロビジョニングする例を紹介します。ここではホスト名の設定を行います。

## 構成図

![architecture](/images/ssmdocument-ec2-provisioning-20240107/architecture.png)

:::message
Ansible を実行するだけであれば、AWS が用意している `AWS-ApplyAnsiblePlaybooks` を利用するのが簡単です。入出力をカスタマイズしたり、自作の SSM オートメーションと組み合わせることを想定しています。
:::

### SSM ドキュメント採用理由

- プロビジョニング用のサーバー (Ansible, Jenkins, etc...) が不要になる
- プロビジョニング実行やログ確認にサーバーへのログインが不要になる

### Ansible 採用理由

- 手作業による誤りや漏れを防止できる
- シェルスクリプトと比べ、冪等性を担保しやすい

### GitHub 採用理由

- Playbook を各サーバーに展開しやすい (`git clone` するだけ)
- 変更内容の確認やレビューがしやすい

## SSM ドキュメント作成

以下リポジトリを参考にしてください。Terraform を利用しています。

https://github.com/namusour0763/ssm_document_sample

https://github.com/namusour0763/ssm_document_sample/blob/main/terraform/modules/ssm_document/ssm_document_apply_ansible.tf

https://github.com/namusour0763/ssm_document_sample/blob/main/terraform/modules/ssm_document/template_apply_ansible.yml

## SSM ドキュメント実行

まずプロビジョニング前の EC2 を確認してみます。セッションマネージャーで接続しホスト名を調べると、自動で設定されるものになっています。

![connect ec2](/images/ssmdocument-ec2-provisioning-20240107/connect_ec2.png)
![hostname before](/images/ssmdocument-ec2-provisioning-20240107/hostname_before.png)

では SSM ドキュメントを実行してみましょう。マネジメントコンソールから作成したドキュメントを選択し「コマンドを実行する」をクリックします。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc1.png)
![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc2.png)

ホスト名の入力とターゲットのインスタンスの選択をします。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc3.png)

ログを S3 ではなく CloudWatch Logs に書き込むよう選択します。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc4.png)

残りはそのままで「実行」をクリックします。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc5.png)

正常に実行できたか確認しましょう。

![ssmdoc result](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_result1.png)
![ssmdoc result](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_result2.png)

ログが少々見にくいので、CloudWatch Logs からも確認してみましょう。

![ssmdoc log](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_log1.png)
![ssmdoc log](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_log2.png)

実際にホスト名が変更できているか確認します。想定通りの結果が得られました！

![hostname after](/images/ssmdocument-ec2-provisioning-20240107/hostname_after.png)

## まとめ

SSM ドキュメントを用いることで、サーバーレスで EC2 を設定することが可能です。このためだけにサーバーを管理する必要がなくなりました！プロビジョニング用にサーバーを増やす前に、一度 Systems Manager の利用を検討してみてください。

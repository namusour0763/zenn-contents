---
title: "Amazon Linux 2023 の AWS CLI をアップデートする方法"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "AWSCLI", "AmazonLinux2023"]
published: true
---

## 前提

- Amazon Linux 2023 にインストール済みの AWS CLI をアップデートしたい
- パッケージ経由でアップデートしたい（インストーラーを使いたくない）
- 他のパッケージを更新したくない

## 解決策

以下コマンドを実行する。

```bash
sudo dnf upgrade aws-cli2 --releasever=latest
```

## 補足

他パッケージも更新してよいなら、以下コマンドで OK。

```bash
sudo dnf upgrade
```

そもそもインストーラーを使ってよいなら、公式ドキュメント通り実施すること。

https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html

`yum` コマンド(`dnf` にエイリアスされているので同じ) でパッケージ版の AWS CLIをアンインストール、zip ファイルを解凍、インストーラーを実行という流れ。
この場合はもともと入っていた `/usr/bin/aws` がなくなり `/usr/local/bin/aws` を使うことになる。

```bash
sudo yum remove awscli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip

# 初回インストール
sudo ./aws/install

# 2回目以降
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
```

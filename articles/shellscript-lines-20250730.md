---
title: "シェルスクリプトでファイルやコマンドの実行結果を1行ずつ処理する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["shell", "linux"]
published: true
---

## やりたいこと

シェルスクリプトでファイルやコマンドの実行結果を1行ずつ処理したい。

## 方法

以下のように `while read` を使えばよい。

### ファイルの各行を処理

```bash
# ファイルの各行を処理
while IFS= read -r line; do
    echo "${line}"
done < test.txt
```

```bash
# 実行結果
[profile A]
credential_source = Ec2InstanceMetadata
role_arn = arn:aws:iam::111111111111:role/RoleA
role_session_name = ProfileARoleSession
[profile B]
credential_source = Ec2InstanceMetadata
role_arn = arn:aws:iam::222222222222:role/RoleB
role_session_name = ProfileBRoleSession
```

:::message
- `IFS=` は文字列の行頭や末尾に存在するタブや空白を消さないために必要
- `read` の `-r` はバックスラッシュをエスケープではなく文字として扱うために必要
:::

### 特定のコマンドの結果を処理

```bash
# 特定のコマンドの結果を処理
grep role_arn test.txt | while IFS= read -r line; do
    AWS_ACCOUNT_ID=$(echo "${line}" | cut -d ':' -f 5)
    echo "AWS Account ID: $AWS_ACCOUNT_ID"
done
```

```bash
# 実行結果
AWS Account ID: 111111111111
AWS Account ID: 222222222222
```

:::message
`cat test.txt | while IFS= read -r line; do` とすると、UUOC (Useless Use of cat) として ShellCheck の指摘が入るため、前述のリダイレクトでの記載を推奨
:::

## ダメな例

空白や改行が存在する場合、`for` では区切られてしまうため、意図しない動作となる。

```bash
# ダメな例
lines=$(cat test.txt)
for line in ${lines}; do
    echo "${line}"
done

# ダメな例2
lines=($(cat test.txt))
for line in "${lines[@]}"; do
    echo "${line}"
done
```

```bash
# 実行結果（どちらも同じ）
[profile
A]
credential_source
=
Ec2InstanceMetadata
role_arn
=
arn:aws:iam::111111111111:role/RoleA
role_session_name
=
ProfileARoleSession
[profile
B]
credential_source
=
Ec2InstanceMetadata
role_arn
=
arn:aws:iam::222222222222:role/RoleB
role_session_name
=
ProfileBRoleSession
```

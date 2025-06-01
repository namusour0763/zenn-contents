---
title: "GitHub Actions を CodeBuild 上で動かす！"
emoji: "🌌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "CodeBuild", "GitHub", "GitHubActions"]
published: true
---

## はじめに

2024/4/24 に GitHub Actions を CodeBuild で動かせるようになったので試してみます！

https://aws.amazon.com/jp/about-aws/whats-new/2024/04/aws-codebuild-managed-github-action-runners/

GitHub Actions 自体の利用方法はこちらをチェック！

https://zenn.dev/namusour0763/articles/github-actions-newbie-20240703

## 何がうれしいのか？

**GitHub Enterprise Server（GitHub のセルフホスト版）の利用者** には特にメリットがあります。GitHub Enterprise では GitHub ホステッドランナーが使えません[^1]。そのため以下のような苦労がありました。

[^1]: https://docs.github.com/ja/enterprise-server@3.11/admin/managing-github-actions-for-your-enterprise/getting-started-with-github-actions-for-your-enterprise/getting-started-with-github-actions-for-github-enterprise-server

- セルフホステッドランナーのプロビジョニングが必要
- OS や エージェントのメンテナンスが大変
- ランナーの可用性や信頼性確保の設計が必要
- ワークフロー非稼働時もサーバー稼働費が発生

セルフホストの苦しみを **マネージドサービスの CodeBuild にオフロードすることが可能** です！

## どうやって利用するのか？

GitHub Actions を利用中であれば、以下手順[^2]で CodeBuild が利用可能です！

[^2]: https://docs.aws.amazon.com/codebuild/latest/userguide/action-runner.html

- GitHub と CodeBuild の接続
- CodeBuild のビルドプロジェクトの作成
- ワークフロー用 yml ファイルの更新

### GitHub と CodeBuild の接続

CodeBuild からビルドプロジェクト作成画面に遷移してください。「ソース」から画面に従って「OAuth」または「個人用アクセストークン」でソースプロバイダに接続します。

![image](/images/github-actions-with-codebuild-20240704/source_connect1.png)
![image](/images/github-actions-with-codebuild-20240704/oauth.png)

:::message
GitHub Enterprise の場合は「個人用アクセストークン」のみの対応です。
![image](/images/github-actions-with-codebuild-20240704/source_connect2.png)
:::

### CodeBuild のビルドプロジェクトの作成

続けてビルドプロジェクトを作っていきましょう。設定項目が多いので、一部のみ記載します。

#### ソース

「ソース」の設定例です。

![image](/images/github-actions-with-codebuild-20240704/source.png)

#### プライマリソースのウェブフックイベント

「プライマリソースのウェブフックイベント」の設定例です。
イベントタイプに「**WORKFLOW_JOB_QUEUED**」を指定してください。

![image](/images/github-actions-with-codebuild-20240704/webhook.png)

#### 環境

「環境」の設定例です。

![image](/images/github-actions-with-codebuild-20240704/env.png)

:::message
GitHub Enterprise への接続が NAT ゲートウェイの EIP などに絞られている場合、コンピューティングに「EC2」を選択し、VPC の設定をしてください。CodeBuild の「Lambda」は残念ながら VPC で起動できません。
:::

#### Buildspec

「Buildspec」はオーバーライドされるため設定不要です。

![image](/images/github-actions-with-codebuild-20240704/buildspec.png)

### ワークフロー用 yml ファイルの更新

ワークフローの yml ファイルの `runs-on:` を以下のように書き換えてください。`<project-name>` はビルドプロジェクト名に変更し、 `${{ github.run_id }}`, `${{ github.run_attempt }}` はそのままで OK です。

```text
runs-on: codebuild-<project-name>-${{ github.run_id }}-${{ github.run_attempt }}
```

#### 具体例

以下のフォルダ構成・ファイルになります。

```text:tree
├── .github
│   └── workflows
│       └── go-test.yml
├── go.mod
├── main.go
└── sqrt
    ├── sqrt.go
    └── sqrt_test.go
```

```yml:go-test.yml
name: Go Test

on:
  pull_request:
    branches: [master]

jobs:
  test:
    name: Run Go Tests
    runs-on: codebuild-github-actions-codebuild-sample-${{ github.run_id }}-${{ github.run_attempt }}

    steps:
      - name: Check out code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: "1.21.3"

      - name: Get dependencies
        run: go mod download

      - name: Run tests
        run: go test -v ./...
```

## 動作確認

プルリクエストを作り、GitHub Actions をトリガーしてみます。

![image](/images/github-actions-with-codebuild-20240704/pr1.png)

正常終了しました。ここのアイコンは CodeBuild ではないようです。

![image](/images/github-actions-with-codebuild-20240704/pr2.png)

では CodeBuild 側を確認してみましょう。こちらも正常終了しています。

![image](/images/github-actions-with-codebuild-20240704/build_result.png)

ログはどうでしょうか？CodeBuild 側にはセルフホストランナーとしてのログのみ記録され、テスト結果は表示されないようです。

![image](/images/github-actions-with-codebuild-20240704/build_log1.png)
![image](/images/github-actions-with-codebuild-20240704/build_log2.png)

ワークフローのログは GitHub Actions 側に記録される、ということですね。そちらを確認してみましょう。想定通り CodeBuild 上で稼働していること、テスト結果が出力されることが確認できました！

![image](/images/github-actions-with-codebuild-20240704/actions_log1.png)
![image](/images/github-actions-with-codebuild-20240704/actions_log2.png)

## まとめ

CodeBuild が GitHub Actions のセルフホステッドランナーに対応しました。コンプライアンスやセキュリティの関係で、GitHub ホステッドランナーが使えない環境での強力な味方になってくれそうです！

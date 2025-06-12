---
title: "Windows で Claude Code はじめてみる"
emoji: "🔰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AI", "Claude", "ClaudeCode"]
published: true
---

## はじめに

Claude Code が Pro プラン（月額 $20）で使えるようになった (2025/6/5) ので、思い切って課金して始めた記録です。まず何をする必要があるのか知りたい人向けです。

## やるべきこと

Windows 環境ではじめる場合のやるべきことリスト（手順）です。

- Claude のアカウント登録
- Claude Pro プランの登録
- WSL2 の有効化
- `node` のインストール
- `claude-code` のインストール
- claude コマンドの実行

### WSL2 の有効化

```powershell
wsl --install
```

https://learn.microsoft.com/ja-jp/windows/wsl/install

### node のインストール

Node.js のバージョン管理をしたいため、Volta を利用しています。

```bash
curl https://get.volta.sh | bash
volta install node
```

https://docs.volta.sh/guide/getting-started

### claude-code のインストール

```bash
npm install -g @anthropic-ai/claude-code
```

https://docs.anthropic.com/ja/docs/claude-code/overview

### claude コマンドの実行

`claude` コマンドを実行し、画面の指示に従ってブラウザからログイン、認証コードをターミナルにコピペします。VS Code を利用してる場合は自動で拡張機能がインストールされます。

### チュートリアル

https://docs.anthropic.com/ja/docs/claude-code/tutorials

意外と簡単なので、ぜひ始めてみましょう！※お試しの人は月額プランであること（デフォルトは年間プラン）をよく確認しましょう！

## 使ってみた

以下手順で時刻変換アプリを作りました。合計 10 分程度でできてしまいびっくりです！

1. 空のディレクトリを作成し `git init` を実行
1. `claude` を起動し `/init` を実行
1. 作りたいアプリの要件をヒアリングしてもらい Markdown に出力
1. `/init` で CLAUDE.md を更新
1. 上記ドキュメントを参照し、作業計画を立てさせる
1. 作業計画に問題がないことを確認し、作業を依頼

https://github.com/namusour0763/next-timezone-converter

画面の装飾が反映されず、これは Claude Code 単体では直せませんでした。Tailwind CSS を v4 から v3 にダウングレードすることで直ったのですが、調査に数十分かかってしまいました。まったく知識がない場合 AI が作成したものの評価や修正ができないので、そこは要注意です。

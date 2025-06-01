---
title: "DevContainer + VSCode で始める Spring Boot"
emoji: "🌸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SpringBoot","Java","VSCode","Devcontainer"]
published: true
---

## はじめに

Spring Boot を VSCode + DevContainer で始める手順の紹介です。事前に以下前提を満たしているか確認してください。

- WSL2 をインストール済み  (Windows の場合)
- Docker Desktop をインストール済み
- VSCode と 拡張機能「Dev Containers」をインストール済み

## Spring Boot 導入手順

### ファイルの作成

VSCode で WSL に接続し、リモート側に拡張機能「Spring Initializr」をインストール。

![alt](/images/springboot-with-devcontainer-20240701/1.png)

`Ctrl + Shift + P` でコマンドパレットを開き「Create a Maven/Gradle project」を実行。ダイアログでいろいろ聞かれるので、入力を進めます。選択する Java のバージョンは後で必要になるのでメモしてください。

![alt](/images/springboot-with-devcontainer-20240701/2.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/a.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/b.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/c.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/d.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/e.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/f.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/g.png =580x)
![alt](/images/springboot-with-devcontainer-20240701/h.png =580x)

作成されたフォルダー（デフォルトは `demo`）のルートに `.devcontainer` フォルダーを作成。それぞれのフォルダー配下に `devcontainer.json`, `settings.json` を作成します。※ `.vscode` は Spring Initializr により作成済み。

![alt](/images/springboot-with-devcontainer-20240701/3.png)

```json:devcontainer.json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/java
{
  "name": "Java",
  // Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
  "image": "mcr.microsoft.com/devcontainers/java:1-21-bullseye",

  "features": {
    "ghcr.io/devcontainers/features/java:1": {}
  },

  // Use 'forwardPorts' to make a list of ports inside the container available locally.
  "forwardPorts": [8080],

  // Use 'postCreateCommand' to run commands after the container is created.
  // "postCreateCommand": "java -version",

  // Configure tool-specific properties.
  "customizations": {
    "vscode": {
      "extensions": [
        "vscjava.vscode-java-pack",
        "vmware.vscode-spring-boot",
        "vscjava.vscode-spring-initializr",
        "oderwat.indent-rainbow",
        "esbenp.prettier-vscode",
        "mhutchie.git-graph"
      ]
    }
  }

  // Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
  // "remoteUser": "root"
}
```

```json:settings.json
{
  // Java 基本設定
  "java.compile.nullAnalysis.mode": "automatic",
  "java.format.settings.url": ".vscode/java-formatter.xml",
  "java.configuration.updateBuildConfiguration": "interactive",

  // エディタ基本設定
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.formatOnPaste": true,
  "editor.formatOnSave": true,
  "editor.formatOnType": true,

  // 各言語ごとの上書き設定
  "[java]": {
    "editor.defaultFormatter": "redhat.java",
    "editor.detectIndentation": false
  },

  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },

  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },

  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

ファイルの内容は自由に変更してください。`devcontainer.json` を最初から作りたい場合、コマンドパレットから「Dev Containers: Configure Container Features...」を実行すると雛形を作ってくれます。

ただし `vscjava.vscode-java-pack`, `vmware.vscode-spring-boot`, `vscjava.vscode-spring-initializr`, をインストールする、コンテナイメージの Java バージョンは Spring Initializr の設定と揃えることをお忘れなく。

### 開発コンテナーを開く

ファイルの準備ができたら、「ファイル > フォルダーを開く」からフォルダーを開きます。するとダイアログが出てくるので「コンテナーで再度開く」をクリック。その他ダイアログは無視してOK。コンテナーの初回起動時は時間がかかる（数分）ので、気長に待ちましょう。

![alt](/images/springboot-with-devcontainer-20240701/4.png)
![alt](/images/springboot-with-devcontainer-20240701/5.png)

開発コンテナーが起動完了し、フォルダーが開いたことを確認。

![alt](/images/springboot-with-devcontainer-20240701/6.png)

`src/main/java/` 配下の `DemoApplication.java` を開き、「▷ Run Java」をクリック。ダイアログに従って「ブラウザーで開く」をクリック。

![alt](/images/springboot-with-devcontainer-20240701/7.png)
![alt](/images/springboot-with-devcontainer-20240701/8.png)

Spring Initializr 実行時に「Spring Security」を入れた場合は「ログイン画面」、入れていない場合は「Whitelabel Error Page」が表示されます。

![alt](/images/springboot-with-devcontainer-20240701/9.png)
![alt](/images/springboot-with-devcontainer-20240701/10.png)

これで Spring Boot 導入完了です！

### Git の設定

Git を使う場合、この設定もしましょう。左ペインから「Initialize Repository」もしくは `git init` コマンドを実行。

![alt](/images/springboot-with-devcontainer-20240701/11.png)

その後 `.gitignore` を開き `.vscode` をコメントアウトします。

![alt](/images/springboot-with-devcontainer-20240701/12.png)

これでリポジトリのコードと設定ファイルが Git で管理可能になります。

### Formatter の設定

フォーマッターも設定しておきましょう。コマンドパレットから「Java: Open Java Formatter Settings with Preview」を実行し「Generate a default profile」をクリック。これで好みのフォーマットを設定できます。

![alt](/images/springboot-with-devcontainer-20240701/13.png)
![alt](/images/springboot-with-devcontainer-20240701/14.png)
![alt](/images/springboot-with-devcontainer-20240701/15.png)

設定ファイルは `.vscode/java-formatter.xml` です。 `settings.json` でフォーマッターを指定しているので、`Ctrl + S` で保存するだけでフォーマットされます。気持ちいい！

![alt](/images/springboot-with-devcontainer-20240701/format.gif)

## おわりに

Eclipse より VSCode が肌に合う方の参考になれば幸いです。ローカル環境を汚さない、環境差分が起こらないのもメリットです！

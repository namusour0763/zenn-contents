---
title: "X-Ray で Lambda のパフォーマンス（ボトルネック関数）を計測してみる"
emoji: "⌛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Lambda", "xray"]
published: true
---

## 概要

X-Ray を用いて Lambda のパフォーマンスを測定し、Python コード内でボトルネックになっている関数を探し出します。試しに動かすだけのため、マネジメントコンソールから各種リソースを作成・設定します。

## やることリスト

- Lambda 関数の作成
- Python コードの作成
- レイヤーの追加
- タイムアウトの設定
- X-Ray の有効化
- Lambda 関数のテスト実行
- X-Ray トレースの確認

## 詳細

### Python コードの作成

以下コードを使用しました。処理の遅い関数が一部存在するケースをシミュレートしています。測定したい関数の前に `@xray_recoder.capture()` を付与します。

```python
import random
import time
import logging
from aws_xray_sdk.core import patch_all
from aws_xray_sdk.core import xray_recorder

patch_all()
logger = logging.getLogger()
logger.setLevel("INFO")

@xray_recorder.capture("emurate_fast_query")
def emurate_fast_query():
    logger.info("start fast query")
    time.sleep(random.randint(10, 100) / 1000)
    logger.info("end fast query")
    return

@xray_recorder.capture("emurate_slow_query")
def emurate_slow_query():
    logger.info("start slow query")
    time.sleep(random.randint(100, 1000) / 1000)
    logger.info("end slow query")
    return

@xray_recorder.capture("emurate_very_slow_query")
def emurate_very_slow_query():
    logger.info("start very slow query")
    time.sleep(random.randint(1000, 3000) / 1000)
    logger.info("end very slow query")
    return

@xray_recorder.capture("normal_function")
def normal_function():
    logger.info("start normal function")
    emurate_fast_query()
    emurate_fast_query()
    emurate_fast_query()
    logger.info("end normal function")
    return

@xray_recorder.capture("slow_function")
def slow_function():
    logger.info("start slow function")
    emurate_very_slow_query()
    emurate_slow_query()
    emurate_very_slow_query()
    logger.info("end slow function")
    return


def lambda_handler(event, context):
    logger.info("start main")
    normal_function()
    slow_function()
    logger.info("end main")
    return
```

### レイヤーの追加

X-Ray の SDK を手早く利用するために `AWSLambdaPowertoolsPython` の AWS レイヤーを追加します。

![image](/images/xray-with-lambda-20250306/layer.png)

### タイムアウトの設定

デフォルトの 3 秒ではタイムアウトするため、タイムアウト値を 30 秒に伸ばします。

![image](/images/xray-with-lambda-20250306/timeout.png)

### X-Ray の有効化

Lambda サービストレースを有効化します。利用する IAM ロールには X-Ray に関する権限が自動で付与されます。

![image](/images/xray-with-lambda-20250306/enable_xray.png)
![image](/images/xray-with-lambda-20250306/iam_role.png)

### Lambda 関数のテスト実行

コード画面から Deploy -> Test を実行します。

![image](/images/xray-with-lambda-20250306/test.png)
![image](/images/xray-with-lambda-20250306/log.png)

### X-Ray トレースの確認

X-Ray のトレースを見てみましょう。 `xray-sample` のうち `slow_functiuon` が占める割合がほとんどであることがわかります。さらにその中で `emurate_very_slow_query` が 2 秒以上処理に時間がかかっていること、この関数が 2 回呼び出されていることがわかります。このようにボトルネックを見つけ、改善につなげることができます。

![image](/images/xray-with-lambda-20250306/trace.png)

## 所感

アプリケーションのボトルネックがグラフィカルにわかるので助かります。ただコードや実行環境に変更が必要になるのでハードルは高いと思いました。もっと簡単な方法があったらコメントお願いします！
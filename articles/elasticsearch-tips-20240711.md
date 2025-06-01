---
title: "Elasticsearch/Kibanaのトラブルシュート＆Tips"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["elasticsearch", "opensearch", "kibana", "aws"]
published: true
---

## はじめに

Elasticsearch + Kibana（実際は AWS の OpenSearch Service + OpenSearch Dashboards）の利用時に遭遇したトラブルシュートや Tips を記録しておきます。

## トラブルシュート＆Tips

### Kibana の画面に接続できない

VPC アクセスの Elasticsearch/Kibana ではインターネットからアクセスできません。以下の対応が必要です。

- パブリックアクセスでドメインを作り直す
- VPC に Windows 踏み台を作る・VPN を張るなどして、VPC 内部からアクセスする

https://docs.aws.amazon.com/ja_jp/opensearch-service/latest/developerguide/vpc.html

### Elasticsearch に接続できない＆ログ転送できない

fluentbit を使う想定です。設定ファイルが正しいか、コメントを確認してください。

```toml:fluent-bit.conf
[OUTPUT]
    Name               es
    Match              apache.access                                              # 対象があっているか
    Host               search-${domain}-${random}.ap-northeast-1.es.amazonaws.com # 接続先があっているか
    Port               443            # ポートが443になっているか 9200ではありません！
    tls                On             # 上記と合わせて必須！
    HTTP_User          ${user_name}   # ユーザー名があっているか 
    HTTP_Passwd        ${password}    # パスワードが合っているか
    Suppress_Type_Name On             # OpenSearch 2.0 以上では必須！
    Index              apache_logs
    Type               _doc
    Time_Key           @timestamp
```

fluent-bit の状態確認方法

```bash
# ステータス確認
systemctl status fluent-bit -l

# ログ確認
journalctl -u fluent-bit -e
```

VPC アクセス・IAM 認証・OpenSearch Serverless など、様々なパターンがあるので、詳細は以下を確認ください。

https://docs.fluentbit.io/manual/pipeline/outputs/elasticsearch

### Kibana でフィールドが認識されない・機能しない

下記エラーが出ていませんか？

- `Unable to filter for presence of meta fields`
- `No cached mapping for this fields. Refresh field list from the Management > Index Patterns page`

この場合は `DashBoards Management` > `Index patterns` > `${your_index}` から「🔃 (Refresh)」をクリックすると解決します。

![image](/images/elasticsearch-tips-20240711/unable_to_filter.png)
![image](/images/elasticsearch-tips-20240711/no_cached_mapping.png)
![image](/images/elasticsearch-tips-20240711/refresh.png)

### Kibana の Visualize で Aggregation にフィールドが出てこない

フィールドの型が Aggregatable でないことが原因です。

同じテキストであっても `keyword` と `text` は以下の通り違いがあります。

- keyword: 文字列全体をひとかたまりで扱う。Aggregation 可。
- text: 文字列をパースして扱う。Aggregation 不可。

![image](/images/elasticsearch-tips-20240711/aggregation.png)
![image](/images/elasticsearch-tips-20240711/fields.png)
![image](/images/elasticsearch-tips-20240711/index_mappings.png)

よって mappings を変更するしかありません。一度作成したインデックスのマッピングは変更できないため、新しいインデックスにマッピングを再作成し reindex するしかありません。またはデータの再投入でも構いません。

https://tech.legalforce.co.jp/entry/2021/12/21/190129

https://shin0higuchi.hatenablog.com/entry/2017/07/03/233833

### Kibana で部分一致検索や数値の比較がしたい

もしできないのであれば、前述と同じデータ型の問題です。ある文字列が `text` でなく `keyword` として扱われていたら部分一致検索はできませんし、数字が `integer` や `long` でなく `keyword` として扱われていたら大小比較できません。

![image](/images/elasticsearch-tips-20240711/search_text.png)

これも mappings を修正して reindex かデータの再投入が必要です。

![image](/images/elasticsearch-tips-20240711/mappings.png)

### Visualize で割合を可視化したい

あるデータの割合やパーセンテージを可視化したい場合、`TSVB (Time Series Visual Builder)` の `Filter Ratio` を利用します。TSVB 以外の Visualize では Filter Ratio が使えません。

![image](/images/elasticsearch-tips-20240711/tsvb.png)

公式ドキュメントがわかりやすいので、要チェックです。

https://www.elastic.co/jp/blog/how-to-display-data-as-a-percentage-in-kibana-visualizations

## 最後に

トラブルがひとつでも解決したら「いいね」をお願いします！

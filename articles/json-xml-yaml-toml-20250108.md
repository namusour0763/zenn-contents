---
title: "JSON XML YAML TOML を見た目で比較してみる"
emoji: "📄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["json", "xml", "yaml", "toml"]
published: false
---

## モチベーション

`JSON`, `XML`, `YAML`, `TOML` について以下ポストが話題になっており、パッとの見た目を比較したいと考えました。

https://x.com/yutkat/status/1876160376003522592

## 比較

同一内容のファイルを各形式で比較してみます。

### JSON

```json
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "age": 28,
      "hobbies": ["reading", "traveling", "swimming"],
      "address": {
        "city": "New York",
        "zipcode": "10001"
      }
    },
    {
      "id": 2,
      "name": "Bob",
      "age": 34,
      "hobbies": ["gaming", "cycling"],
      "address": {
        "city": "San Francisco",
        "zipcode": "94105"
      }
    }
  ]
}
```

### XML

```xml
<root>
  <users>
    <user>
      <id>1</id>
      <name>Alice</name>
      <age>28</age>
      <hobbies>
        <hobby>reading</hobby>
        <hobby>traveling</hobby>
        <hobby>swimming</hobby>
      </hobbies>
      <address>
        <city>New York</city>
        <zipcode>10001</zipcode>
      </address>
    </user>
    <user>
      <id>2</id>
      <name>Bob</name>
      <age>34</age>
      <hobbies>
        <hobby>gaming</hobby>
        <hobby>cycling</hobby>
      </hobbies>
      <address>
        <city>San Francisco</city>
        <zipcode>94105</zipcode>
      </address>
    </user>
  </users>
</root>
```

### YAML

```yaml
users:
  - id: 1
    name: Alice
    age: 28
    hobbies:
      - reading
      - traveling
      - swimming
    address:
      city: New York
      zipcode: "10001"
  - id: 2
    name: Bob
    age: 34
    hobbies:
      - gaming
      - cycling
    address:
      city: San Francisco
      zipcode: "94105"
```

### TOML

```toml
[[users]]
id = 1
name = "Alice"
age = 28
hobbies = ["reading", "traveling", "swimming"]

[users.address]
city = "New York"
zipcode = "10001"

[[users]]
id = 2
name = "Bob"
age = 34
hobbies = ["gaming", "cycling"]

[users.address]
city = "San Francisco"
zipcode = "94105"
```

### データで見てみる

| ファイル形式 | 行数 | 文字数(スペース込み) |
| ------------ | ---- | -------------------- |
| JSON         | 24   | 417                  |
| XML          | 31   | 623                  |
| YAML         | 20   | 307                  |
| TOML         | 19   | 270                  |

## 所感

- `JSON`, `YAML` はデータ構造が見やすい
- `XML` はどうしても見た目がゴツくなる
- `TOML` は行数・文字数ともに最小
  - ただしテーブルの配列やネストが多いとツラそう

基本は `JSON`, レガシーシステムなら `XML`, 簡潔にしたいなら `YAML`, テーブルの配列やネストが絶対にないなら `TOML` が最適かなと考えます。

## 参考

https://toml.io/ja/v1.0.0

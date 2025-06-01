---
title: "Apache のダミーログを作る！"
emoji: "🌳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["apache", "go"]
published: true
---

## はじめに

ログ転送やログ分析の検証の際、ダミーのログがほしくなりました。そこで Apache のアクセスログ形式でログを出力するプログラムを Go で作成します。

## 要件

- **Apache のアクセスログ (Combined Log Format) を標準出力に出力する**
- **ログ出力の内容を重み付けできる**
- 1 秒あたりのログ出力量を制御できる
- ログ出力の内容確認や変更が簡単である
- 外部ツールやライブラリの導入が不要である

完全ランダムなログ出力だと分析が面白くないため、重み付けができることを要件としています。例えば `/login` へのアクセスが非常に多く、すべて `5xx` エラーが返ってくる、などのケースをシミュレートするためです。

## コード

```go:main.go
package main

import (
  "fmt"
  "math/rand"
  "os"
  "strconv"
  "strings"
  "time"
)

type WeightedItem struct {
  item   string
  weight int
}

type LogConfig struct {
  ipAddresses      []WeightedItem
  endpointStatuses []WeightedItem
  userAgents       []WeightedItem
}

var logConfig = LogConfig{
  ipAddresses: []WeightedItem{
    {item: "192.168.1.1", weight: 50},
    {item: "10.0.0.1", weight: 30},
    {item: "172.16.0.1", weight: 10},
    {item: "8.8.8.8", weight: 5},
    {item: "1.1.1.1", weight: 5},
  },
  endpointStatuses: []WeightedItem{
    {item: "/,200", weight: 40},
    {item: "/about,200", weight: 10},
    {item: "/contact,200", weight: 10},
    {item: "/products,200", weight: 10},
    {item: "/products,404", weight: 5},
    {item: "/services,200", weight: 5},
    {item: "/login,500", weight: 100},
  },
  userAgents: []WeightedItem{
    {item: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36", weight: 100},
    {item: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15", weight: 30},
    {item: "Mozilla/5.0 (X11; Linux x86_64; rv:89.0) Gecko/20100101 Firefox/89.0", weight: 20},
  },
}

func weightedRandomChoice(items []WeightedItem) string {
  totalWeight := 0
  for _, item := range items {
    totalWeight += item.weight
  }

  randomNumber := rand.Intn(totalWeight)
  for _, item := range items {
    randomNumber -= item.weight
    if randomNumber < 0 {
      return item.item
    }
  }

  return items[0].item // This should never happen, but it's here as a fallback
}

func generateLogEntry() string {
  ipAddress := weightedRandomChoice(logConfig.ipAddresses)
  endpointStatus := weightedRandomChoice(logConfig.endpointStatuses)
  userAgent := weightedRandomChoice(logConfig.userAgents)

  parts := strings.Split(endpointStatus, ",")
  endpoint := parts[0]
  statusCode := parts[1]

  timestamp := time.Now().Format("02/Jan/2006:15:04:05 -0700")
  method := "GET"
  protocol := "HTTP/1.1"
  bytesSent := rand.Intn(5000) + 500
  referer := "-"

  return fmt.Sprintf("%s - - [%s] \"%s %s %s\" %s %d \"%s\" \"%s\"",
    ipAddress, timestamp, method, endpoint, protocol, statusCode, bytesSent, referer, userAgent)
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("Usage: go run main.go <lines_per_second>")
    os.Exit(1)
  }

  linesPerSecond, err := strconv.Atoi(os.Args[1])
  if err != nil || linesPerSecond <= 0 {
    fmt.Println("Please provide a valid positive integer for lines per second")
    os.Exit(1)
  }

  ticker := time.NewTicker(time.Second)
  defer ticker.Stop()

  for range ticker.C {
    for i := 0; i < linesPerSecond; i++ {
      fmt.Println(generateLogEntry())
    }
  }
}
```

各データのエントリーを増やしたり、method や protocol も選択肢から選べるようにするなど、要件に合わせた改造がしやすい作りです。

## 実行

```bash
# 1 秒あたり 5 行のログを出力
go run main.go 5

# 出力内容
10.0.0.1 - - [11/Jul/2024:14:38:13 +0000] "GET /login HTTP/1.1" 500 1541 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
10.0.0.1 - - [11/Jul/2024:14:38:13 +0000] "GET /login HTTP/1.1" 500 2880 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15"
10.0.0.1 - - [11/Jul/2024:14:38:13 +0000] "GET / HTTP/1.1" 200 3731 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
192.168.1.1 - - [11/Jul/2024:14:38:14 +0000] "GET /login HTTP/1.1" 500 4054 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
172.16.0.1 - - [11/Jul/2024:14:38:14 +0000] "GET /login HTTP/1.1" 500 1355 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"

# Ctrl + C で停止
^Csignal: interrupt
```

## その他

より高度なダミーログ出力が必要な場合、以下モジュールの活用も検討ください。

https://github.com/mingrammer/flog

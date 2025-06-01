---
title: "暗号化・ハッシュ関数の処理速度を Go で比べてみた"
emoji: "🔒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["暗号","ハッシュ","Go"]
published: true
---


## はじめに

ユーザーのパスワード保存について調べていたところ、「Bcrypt などは意図的に遅くしている」との記載を見つけました。Go を用いてどれくらい遅いのか実際に測ってみます。

https://github.com/namusour0763/go-crypt-comparison

## コード

MD5, SHA-256, Bcrypt, Scrypt について 100 回処理を行い、平均処理時間を計測します。

```go
package main

import (
  "crypto/md5"
  "crypto/sha256"
  "fmt"
  "time"

  "golang.org/x/crypto/bcrypt"
  "golang.org/x/crypto/scrypt"
)

func main() {
  password := "password" // パスワード
  iterations := 100      // 計算回数

  // MD5
  md5Duration := benchmark(iterations, func() {
    md5.Sum([]byte(password))
  })

  // SHA-256
  sha256Duration := benchmark(iterations, func() {
    sha256.Sum256([]byte(password))
  })

  // Bcrypt
  bcryptDuration := benchmark(iterations, func() {
    _, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
      fmt.Printf("Bcryptエラー: %v\n", err)
    }
  })

  // Scrypt
  scryptDuration := benchmark(iterations, func() {
    _, err := scrypt.Key([]byte(password), []byte("salt"), 32768, 8, 1, 32)
    if err != nil {
      fmt.Printf("Scryptエラー: %v\n", err)
    }
  })

  fmt.Printf("MD5の平均時間 (%d回): %v\n", iterations, md5Duration)
  fmt.Printf("SHA-256の平均時間 (%d回): %v\n", iterations, sha256Duration)
  fmt.Printf("Bcryptの平均時間 (%d回): %v\n", iterations, bcryptDuration)
  fmt.Printf("Scryptの平均時間 (%d回): %v\n", iterations, scryptDuration)
}

func benchmark(iterations int, hashFunc func()) time.Duration {
  var totalDuration time.Duration
  for i := 0; i < iterations; i++ {
    start := time.Now()
    hashFunc()
    totalDuration += time.Since(start)
  }
  return totalDuration / time.Duration(iterations)
}
```

## 実行結果

実行環境

- CPU: Core-i7 6700K @ 4GHz
- Memory: 16 GB
- OS: Ubuntu 20.04 LTS (on WSL2)
- Go: version 1.21.3

実行結果

| ハッシュ関数 | 実行時間 (100回平均) |
| ------------ | -------------------- |
| MD5          | 0.139 ms             |
| SHA-256      | 0.221 ms             |
| Bcrypt       | 55.2 ms              |
| Scrypt       | 76.7 ms              |

以下のことが分かりました。

- Bcrypt は 数十ミリ秒程度の処理時間である
- Bcrypt は MD5 や SHA-256 に比べ数百倍（２桁）遅い

## その他

上記コードは Claude 3.5 Sonnet に質問をふたつ投げ込んだだけ、数十秒で完成しました（パラメーターや順序入れ替えなどはしています）。生成 AI をうまく使えるようになると、効率が爆上がりしそうです！

![alt](/images/crypt-method-comparison-20240630/claude.png)

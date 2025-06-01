---
title: "Javaを難しく感じる理由を言語化してみる"
emoji: "☕"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Java", "SpringBoot"]
published: true
---

## はじめに

### 記事の目的

まず **Java 批判ではありません。** 次に **Java 固有の問題ではありません。** 筆者は Java によるアプリケーション作成が難しいという苦手意識を持っています。所属会社は SIer であり、Java による開発は花形です。せっかくなら Java に強くなりたいと思い、この問題を乗り越えるため思ったことを言語化しようという記事です。

### 筆者について

- AWS メインのインフラエンジニア
- 業務では Terraform, Python, ShellScript を利用
- アプリケーション開発はほぼ経験無し
- Java は Spring + MyBatis を 1 週間程度研修でやった程度

上記バックグラウンドのため、「Java を難しく感じる理由」とは言語固有の問題ではなく、 **筆者の業務との乖離やアプリケーション開発の難しさに起因している** 部分が多いと考えられます。

## Java を難しく感じる理由

### コードが長い

1 行の文字数、また行数が多いと感じます。単純に分量が多く、パッと見たとき「うっ、ゴツい。」と思います。同じような記載の繰り返しが多いなあと思うことも。

:::details Java のコード例

```xml:pom.xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

```java:Employee.java
package com.example.demo.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String department;

    // Getters and setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }
}
```

```java:EmployeeService.java
package com.example.demo.service;

import com.example.demo.entity.Employee;
import com.example.demo.repository.EmployeeRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    public List<Employee> getAllEmployees() {
        return employeeRepository.findAll();
    }

    public Optional<Employee> getEmployeeById(Long id) {
        return employeeRepository.findById(id);
    }

    public Employee addEmployee(Employee employee) {
        return employeeRepository.save(employee);
    }

    public Employee updateEmployee(Long id, Employee employeeDetails) {
        Optional<Employee> optionalEmployee = employeeRepository.findById(id);
        if (optionalEmployee.isPresent()) {
            Employee employee = optionalEmployee.get();
            employee.setName(employeeDetails.getName());
            employee.setDepartment(employeeDetails.getDepartment());
            return employeeRepository.save(employee);
        } else {
            throw new RuntimeException("Employee not found with id " + id);
        }
    }

    public void deleteEmployee(Long id) {
        employeeRepository.deleteById(id);
    }
}
```

:::

プログラミング言語の構文の表現力について調査があります。Wikipedia からの抜粋ですが、C 言語と比較すると Java は 命令文・行数において 1.5~2.5 倍 効率的なようです。一方 Python は 6~6.5 倍となっています。

| 言語 | 文の数の比 | 行数の比 |
| ------ | --- | --- |
| C      | 1   | 1   |
| Java   | 2.5 | 1.5 |
| Python | 6   | 6.5 |

https://ja.wikipedia.org/wiki/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E8%A8%80%E8%AA%9E%E3%81%AE%E6%AF%94%E8%BC%83

ただし Java には Stream API などの強力な機能もついており、常に他の言語より長くなるわけではないようです。

:::details Stream API の例

「与えられた商品群の中から最も価格の高い商品を返す関数」の実装例です。
以下のリンクから引用しています。

```java:java
public Optional<Item> getMostExpensiveItem(List<Item> items) {
    return items.stream().max(Comparator.comparing(Item::getPrice));
}
```

```go:go
func getMostExpensiveItem(items []*Item) *Item {
    if len(items) == 0 {
        return nil
    }

    maxPrice := items[0].price
    maxPriceItem := items[0]

    for _, item := range items {
        if price := item.price; price > maxPrice {
            maxPrice = price
            maxPriceItem = item
        }
    }

    return maxPriceItem
}
```

:::

https://inside.dmm.com/articles/ex-java-engineer-thoughts-about-go/

### ファイルが多い

`pom.xml`、`application.properties`、エンティティクラス、リポジトリーインターフェース、サービスクラス、コントローラークラス、etc... とファイルが非常に多く混乱します。どのパッケージやファイルを見ればいいのか、どこに何を書くべきなのかが分からず、物量に圧倒されてしまいます。

![alt](/images/think-about-java-20240609/files_blur.png =200x)
*先生！どのファイルを見ればいいんでしょうか！*

### 必要な前提知識が多い

Java に限った話ではないですが、アプリケーションを作るためにはその言語を知っているだけでは不十分です。フレームワーク、バリデーション、データベース、SQL、認証認可、etc... と様々な要素を要求されます。必要な前提知識が多く、分からないところが分からない、となりがちです。

### すぐに使えない

シェルや Python は Windows (WSL2), Linux, Mac ですぐに利用することが出来ます。一方 Java は JDK をインストールしたりパスを通したりが必要で、スッと使い始められません。プロジェクトやフレームワークの設定も必要だったりします。

この点は Pleiades All in One によって解決できることが分かったので、初心者はこれを利用しましょう！

https://willbrains.jp/

### Eclipse による開発が前提

多くの書籍や研修が Eclipse による開発を前提にしています。Eclipse は エディタではなく IDE であり、多機能がゆえ難しいです。

私は VSCode と devcontainer や 拡張機能周りのエコシステムが好きなのですが、Java に関しては実例が少なくトラブルにハマる可能性も高いのでやめました。

:::message
Eclipse が重いというのは偏見でした。SIer 御用達の PC ではキツかったですが、私用 PC であれば VSCode ほどではないにしろ軽快に動きました。
:::

![alt](/images/think-about-java-20240609/eclipse_blur.png =400x)
*初心者を圧倒する多機能な画面！*

## まとめ

タイトル詐欺のようですが、Java の難しさというよりアプリケーション作成の難しさなんだろうな、と言語化することで気付かされました。Next.js で簡単なブログを作ったことがあるのですが、そこでも実感する部分はありました。上記の苦手意識を克服するにはどうしたらよいか、有識者に相談してみようと思います。

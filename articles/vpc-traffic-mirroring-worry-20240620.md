---
title: "VPCトラフィックミラーリングの将来が気になる件"
emoji: "🪞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "VPC", "EC2"]
published: true
---

## VPCトラフィックミラーリングとは

VPC Traffic Mirroring とは、2019 年 6 月にリリースされた、ネットワークトラフィックを捉えて検査することができる VPC の機能です。スイッチングハブやルーターのミラーポートと同等のものといえます。

https://aws.amazon.com/jp/blogs/news/new-vpc-traffic-mirroring/


https://www.imrcry.jp/blog/packet-capture/334/

## VPCトラフィックミラーリングの用途

主な用途はネットワークのセキュリティやコンプライアンスの実装です。

VPC フローログでは通信の流れしか確認することができないため、セキュリティインシデント発生時に解析が難しい可能性があります。一方 VPC トラフィックミラーリングではパケットをキャプチャするため、通信の中身を確認することが可能です。

例えば、以下の Arkime と組み合わせると、フォレンジックにより攻撃内容や流出した情報などがより正確に確認できます。

https://arkime.com/

https://blog.nicter.jp/2023/03/arkime_in_secops/

## VPCトラフィックミラーリングの将来が気になる点

![alt](/images/vpc-traffic-mirroring-worry-20240620/limitations.png)
*2024/06/20 時点での VPC トラフィックミラーリングにおける制約*

https://docs.aws.amazon.com/vpc/latest/mirroring/traffic-mirroring-limits.html

上記は Traffic Mirroring における制約なのですが、なんと **比較的新しい[^1] EC2 インスタンスタイプではまったく使えません。** ミラーリングしたい場合、M5 や M4 といった古いインスタンスタイプを使い続けるしかありません。

[^1]: ※新しいといいますが、例えば M6 インスタンスタイプは 2021 年 8 月リリースなので 3 年近く経っています。対応する気がない…？

古いタイプで使えないならわかるのですが新しいタイプで使えないとなると「AWS が意図的に対応していない？」「将来使えなくなる機能なのでは…？」と疑ってしまいます。

## 感想

VPC トラフィックミラーリングが使えなくなる未来が現実とならないよう祈っています。

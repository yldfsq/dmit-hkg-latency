# DMIT 香港测速：三条线路的测试 IP、Looking Glass 走法，以及下单前该看懂的延迟与丢包数据

搜「DMIT 香港测速」的人，多半不是想听品牌介绍。你手上大概已经有一串候选商家，现在只想知道一件事：这台香港机器连到我家宽带到底快不快、稳不稳。

问题在于，DMIT 的香港机房不是一个产品，而是一整套按线路切开的套餐线。同一个机房、同样的 AMD EPYC 硬件，Pro 和 Tier 1 测出来的数字能差一倍以上。拿一个 IP 测完就下结论，很容易买错。

这篇文章把测速这件事拆开讲：测哪些 IP、用什么工具、第三方实测数据长什么样、数字难看的几种常见原因，以及测完之后怎么对应到具体套餐。

## DMIT 香港三条线路，测出来的数字完全不是一回事

DMIT 香港（HKG）目前主要有三类网络，区别不在 CPU 和硬盘，而在路由走法：

| 线路系列 | 路由特征 | 中国大陆访问 | 典型用途 |
| --- | --- | --- | --- |
| HKG.Pro（Premium） | 三网回程走香港 CTG 接入国内 CN2 GIA；去程电信 CN2 GIA 直连、联通 AS4837 + AS10099、移动 CMI | 优化最好 | 延迟敏感业务、跨境中转、实时接口 |
| HKG.EB（Eyeball） | 电信去程 CN2，回程 163；移动 CMI 双向；联通去程绕日本 NTT | 中等，移动体验最好 | 建站、预算有限的国内访问需求 |
| HKG.T1（Tier 1） | 国际线路，接入 RETN、NTT 等，无大陆专项优化 | 基本没有 | 面向海外用户、大带宽、落地用 |

另外还有一条更便宜的 **Standard 系列**（原来的 Lite），走的是大带宽路线，回程移动 CMI，1Gbps 起步，适合不追求低延迟但流量吃得多的场景。

关键点：这三条线路的**测试 IP 是分开的**。用 Pro 的 IP 测出 40ms，不代表你买的 T1 也是 40ms。

## DMIT 香港测试 IP 与 Looking Glass 节点一览

DMIT 官方提供 Looking Glass（简称 LG），根地址是 `lg.dmit.sh`，可以按机房和线路切换节点，直接跑 Ping、Traceroute 和 MTR。以下测试 IP 和节点编号在多家测评站和主机资讯站的整理中是一致的：

| 节点 | 测试 IPv4 | Looking Glass |
| --- | --- | --- |
| 香港 Pro（Premium / CN2 GIA） | 103.117.100.199 | `lg.dmit.sh/?server=5` |
| 香港 Eyeball（EB / 三网 CMI） | 154.12.188.3 | `lg.dmit.sh/?server=6` |
| 香港 Tier 1（国际线路） | 154.12.176.28 | `lg.dmit.sh/?server=7` |

顺手记一下：LG 的 server 编号在不同时期有过调整，如果 `server=5` 打开后显示的机房不对，直接在根页面 `lg.dmit.sh` 的下拉菜单里选「Hong Kong」对应的节点。

## 十分钟做完一次有用的测速

只 Ping 一次就下单，是买 VPS 最常见的踩坑方式。完整的流程是三步。

**第一步：本机 Ping 测试 IP，看去程。**

Windows 用 `ping 103.117.100.199 -t`，Mac / Linux 用 `ping 103.117.100.199`。跑个一两分钟，重点不是最快的那一个数值，而是**平均延迟**和**丢包率**。香港节点连华南地区，20–30ms 属于低；华北、华东 40–60ms 也正常。丢包率 0% 是理想值。

**第二步：打开 LG，测回程路由。**

这是 DMIT 这个工具真正比 `ping` 有用的地方——`ping` 只能看到「你 → 服务器」这一段，回程走什么线路，只能从服务器那侧往外跑。

操作是：进 LG 页面 → 选对应节点 → 命令选 **Traceroute** → 目标 IP 填你自己的公网 IP（不确定就去 ip.sb 查一下）→ 运行。

看到结果后，认几个 AS 号就够了：

- **AS4809 / AS23764** —— 中国电信 CN2 GIA，高端线路，出现它说明走的是一条好路
- **AS9929** —— 联通精品网（AS10099 同属联通的国际段）
- **AS58453 / AS9808** —— 移动 CMI
- **AS4134 / AS163** —— 电信普通 163 骨干，比 CN2 差一档
- **AS4837** —— 联通普通线路

如果你买的是 Pro 套餐，回程出现 59.43 开头的节点基本对得上；如果出现 202.97 之类的普通 163 节点，说明路由和宣传不一致，可以拿这个结果去开工单。

**第三步：MTR 定位丢包。**

Ping 有丢包但不知道丢在哪，就跑一次 **MTR**。它能逐跳显示丢包率，一眼看出是本地宽带的问题、跨境段的问题，还是机房侧的问题。

## 第三方实测数据：香港 Pro 大约 43ms，EB 大约 57ms

DMIT 自己不放"官方跑分"，所以数字都来自第三方测评。综合目前能查到的几份报告：

> **香港 Premium（Pro）**：多个测评平台测出的全国三网平均 Ping 约 43ms，电信约 42ms、联通约 42ms、移动约 46ms，日常丢包接近零。另一份 CN2 GIA 专项评测里，国内平均延迟约 45ms，500Mbps 带宽实测跑出接近 600Mbps，硬盘 I/O 平均约 1GB/s。
>
> **香港 Eyeball（EB）**：一份针对 HKG.EB 的详细评测给出中国大陆平均延迟约 57ms。电信去程走 CN2（59.43 段）直连香港，回程落到电信 163（AS4134）；移动是 CMI 双向直连，体验最好；联通去程绕日本，延迟相对高。晚高峰大包回程在联通、移动的部分节点出现了丢包，测评点之间差异较大。
>
> **香港 Tier 1（T1）**：4Gbps 端口，内地节点下载基本能跑到 1Gbps 左右，港台节点上传超过 3Gbps。但电信去程可能先绕到欧洲再到香港，联通绕日韩，只有移动是直连。电信用户测这台机器，延迟可能比美国节点还高。

硬件层面倒是差别不大：Pro 与 EB 的新套餐已升级到 AMD EPYC 9005 系列（有测评显示 EB 实机使用 EPYC 9655P，主频 2.60GHz），SSD 读写稳定在 1.1GB/s 左右。同一个系列的机器，决定体验的是路由，不是配置单。

## 测速结果怎么看：三个容易看错的地方

**绕路不等于丢包。** 延迟高和丢包是两个问题。绕到荷兰再回香港的电信去程，延迟会很难看，但链路本身可能是稳的。这种情况的结论应该是"这条线路不适合电信"，而不是"这机器不行"。

**晚高峰才见真章。** 白天测出来的 40ms 和晚上八点测出来的 40ms 不是一个概念。至少在工作日 20:00–23:00 各测一次，看延迟抖动幅度和丢包。上面 EB 的实测里，联通和移动在晚高峰出现丢包，白天测是发现不了的。

**一次结果不算数。** DMIT 官方目前仍把香港 Eyeball 标注为 Beta 阶段，产品和路由在持续调整。任何单次测速都只是当下的快照，不等于长期线路承诺。

## 测完再决定买哪档：DMIT 香港在售套餐与价格

以下配置和价格整理自 DMIT 官网产品页与多方测评站的公开信息。库存和价格变动频繁，实际下单时请以官网结算页显示为准。所有购买入口统一指向同一推广页。

### HKG.T1（国际线路，年付 $36.9 起）

| 套餐 | CPU | 内存 | 硬盘 | 流量 / 带宽 | 价格 | 购买 |
| --- | --- | --- | --- | --- | --- | --- |
| HKG.T1.WEE | 1 核 | 1 GB | 20 GB SSD | 1 TB / 4 Gbps | $36.9/年 | [ 查看 T1 WEE 年付方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.TINY | 1 核 | 1 GB | 20 GB SSD | 2 TB / 4 Gbps | $6.9/月，$76.9/年 | [ 查看 T1 TINY 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.STARTER | 1 核 | 2 GB | 40 GB SSD | 4 TB / 10 Gbps | $12.9/月，$154.8/年 | [ 查看 T1 STARTER 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.MINI | 2 核 | 2 GB | 60 GB SSD | 8 TB / 10 Gbps | $21.9/月，$262.8/年 | [ 查看 T1 MINI 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.MICRO | 4 核 | 4 GB | 80 GB SSD | 16 TB / 10 Gbps | $32.9/月 | [ 查看 T1 MICRO 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.MEDIUM | 4 核 | 8 GB | 160 GB SSD | 32 TB / 10 Gbps | $49.9/月 | [ 查看 T1 MEDIUM 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.LARGE | 8 核 | 16 GB | 320 GB SSD | 64 TB / 10 Gbps | $99.9/月 | [ 查看 T1 LARGE 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.GIANT | 8 核 | 24 GB | 640 GB SSD | 128 TB / 10 Gbps | $199.9/月 | [ 查看 T1 GIANT 方案](https://www.dmit.io/aff.php?aff=12046) |

T1 系列的特点是**流量跑超不会断线**，而是降到限速档继续用（WEE 限 50Mbps，中高配最高 1Gbps）。对月底流量焦虑的人来说，这个设计比多给 2TB 流量有用。

### HKG.Pro（三网 CN2 GIA，月付 $39.9 起）

Pro 常规产品已从 AS3 升级到 AN5 平台（AMD EPYC 9005 + DDR5），价格没变。

| 套餐 | CPU | 内存 | 硬盘 | 流量 / 带宽 | 价格 | 购买 |
| --- | --- | --- | --- | --- | --- | --- |
| HKG.Pro.TINY | 1 核 | 1 GB | 20 GB SSD | 500 GB / 1 Gbps | $39.9/月 | [ 查看 Pro TINY 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.STARTER | 1 核 | 2 GB | 40 GB SSD | 1 TB / 1 Gbps | $79.9/月 | [ 查看 Pro STARTER 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MINI | 2 核 | 2 GB | 60 GB SSD | 1.5 TB / 1 Gbps | $119.9/月 | [ 查看 Pro MINI 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MICRO | 2 核 | 4 GB | 80 GB SSD | 2 TB / 1 Gbps | $159.9/月 | [ 查看 Pro MICRO 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MEDIUM | 4 核 | 4 GB | 160 GB SSD | 2.5 TB / 1 Gbps | $179.9/月 | [ 查看 Pro MEDIUM 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.LARGE | 4 核 | 8 GB | 240 GB SSD | 3 TB / 1 Gbps | $239.9/月 | [ 查看 Pro LARGE 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.GIANT | 8 核 | 16 GB | 320 GB SSD | 6 TB / 1 Gbps | $499.9/月 | [ 查看 Pro GIANT 方案](https://www.dmit.io/aff.php?aff=12046) |

Pro 是香港延迟最低的一档，也是最贵的一档。如果是个人博客或轻量站点，说实话不太值——同样预算在 EB 系列能拿到 3 倍以上流量。

### HKG.EB（Eyeball，AS3 与 AN5 双平台）

EB 目前分 AS3（EPYC 7003）和 AN5（EPYC 9005 + DDR5 ECC）两个硬件平台，网络线路一致。另外还有 WEE 这类入门特价档会不定期补货，例如 1 核 1G / 20G / 500GB 流量 / 500Mbps 的 HKG.AS3.EB.WEE，价格 $24.9/月。

**HKG.AS3.EB：**

| 套餐 | CPU | 内存 | 硬盘 | 流量 / 带宽 | 价格 | 购买 |
| --- | --- | --- | --- | --- | --- | --- |
| HKG.AS3.EB.WEE | 1 核 | 1 GB | 20 GB SSD | 500 GB / 500 Mbps | $24.9/月 | [ 查看 EB WEE 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.TINY | 1 核 | 1 GB | 20 GB SSD | 800 GB / 1 Gbps | $39.9/月 | [ 查看 EB TINY 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.STARTER | 1 核 | 2 GB | 40 GB SSD | 1.5 TB / 1 Gbps | $79.9/月 | [ 查看 EB STARTER 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.MINI | 2 核 | 4 GB | 60 GB SSD | 2.2 TB / 1 Gbps | $126.9/月 | [ 查看 EB MINI 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.MICRO | 4 核 | 4 GB | 80 GB SSD | 3 TB / 1 Gbps | $179.9/月 | [ 查看 EB MICRO 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.EB.MEDIUM | 4 核 | 8 GB | 160 GB SSD | 4 TB / 1 Gbps | $239.9/月 | [ 查看 EB MEDIUM 方案](https://www.dmit.io/aff.php?aff=12046) |

**HKG.AN5.EB：**

| 套餐 | CPU | 内存 | 硬盘 | 流量 / 带宽 | 价格 | 购买 |
| --- | --- | --- | --- | --- | --- | --- |
| HKG.AN5.EB.TINY | 4 核 | 4 GB | 80 GB SSD | 2.2 TB / 1 Gbps | $149.9/月 | [ 查看 AN5 EB TINY 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.STARTER | 4 核 | 4 GB | 160 GB SSD | 3 TB / 1 Gbps | $199.9/月 | [ 查看 AN5 EB STARTER 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.MINI | 6 核 | 8 GB | 160 GB SSD | 4 TB / 1 Gbps | $279.9/月 | [ 查看 AN5 EB MINI 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.MICRO | 8 核 | 16 GB | 320 GB SSD | 4.5 TB / 1 Gbps | $359.9/月 | [ 查看 AN5 EB MICRO 方案](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.EB.MEDIUM | 12 核 | 24 GB | 640 GB SSD | 9 TB / 1 Gbps | $759.9/月 | [ 查看 AN5 EB MEDIUM 方案](https://www.dmit.io/aff.php?aff=12046) |

AN5 平台起步就是 4 核 4G，价格直接跳到 $149.9/月，适合数据库、API 后端这类吃单核性能和响应速度的业务，个人用户基本用不到。

### HKG Standard（原 Lite，大带宽路线，月付 $12.9 起）

这条线原来叫 Lite，现已更名 Standard。最低配的 $6.9/月 档已经下架，目前起步是 $12.9/月；也有渠道显示入门档调价后为 $7.9/月起（1 核 1G / 10G SSD / 1TB）。三网回程走移动 CMI，去程据用户反馈已调整为 CN2。

| 套餐 | CPU | 内存 | 硬盘 | 流量 / 带宽 | 价格 | 购买 |
| --- | --- | --- | --- | --- | --- | --- |
| Standard STARTER | 1 核 | 1.5 GB | 20 GB SSD | 4 TB / 1 Gbps | $12.9/月 | [ 查看 Standard STARTER 方案](https://www.dmit.io/aff.php?aff=12046) |
| Standard MINI | 2 核 | 2 GB | 40 GB SSD | 6 TB / 1 Gbps | $16.9/月 | [ 查看 Standard MINI 方案](https://www.dmit.io/aff.php?aff=12046) |
| Standard MICRO | 2 核 | 4 GB | 40 GB SSD | 8 TB / 1 Gbps | $21.9/月 | [ 查看 Standard MICRO 方案](https://www.dmit.io/aff.php?aff=12046) |
| Standard MEDIUM | 4 核 | 4 GB | 60 GB SSD | 12 TB / 1 Gbps | $32.9/月 | [ 查看 Standard MEDIUM 方案](https://www.dmit.io/aff.php?aff=12046) |
| Standard LARGE | 4 核 | 8 GB | 100 GB SSD | 22 TB / 1 Gbps | $49.9/月 | [ 查看 Standard LARGE 方案](https://www.dmit.io/aff.php?aff=12046) |
| Standard GIANT | 8 核 | 16 GB | 200 GB SSD | 44 TB / 1 Gbps | $99.9/月 | [ 查看 Standard GIANT 方案](https://www.dmit.io/aff.php?aff=12046) |

## 现在还能用的优惠码

DMIT 的优惠码按产品线和付款周期分开，基本全是**循环折扣**（续费同价），这点比首年折扣良心。目前公开整理里出现频率较高的有：

| 优惠码 | 适用范围 | 折扣 |
| --- | --- | --- |
| `202510_HKG_TYO_PRO_20OFF_RECURRING` | 香港 / 东京 Pro，季付及以上 | 8 折，循环 |
| `202510_HKG_TYO_T1_30OFF_RECURRING` | 香港 / 东京 T1，季付及以上（不含 WEE） | 7 折，循环 |
| `Lite-Annually-Recur-30OFF` | Standard（原 Lite）系列年付 | 7 折，循环 |
| `Lite-Semi-Annually-Recur-20OFF` | Standard 系列半年付 | 8 折，循环 |
| `HKG-EB-V2-ANNUALLY-15OFF-RECUR` | EB TINY 及以上年付 | 85 折，循环 |

优惠码会随活动和库存轮换，也可能随时失效。用法是在结算页选好套餐和付款周期后填入，点 `Validate Code` 验证通过才算生效——不验证直接下单，很可能原价付款。另外不同码对付款周期有硬性要求，拿年付码去套月付是过不了的。

## 关于 DMIT 香港测速的常见问题

**测速 IP 和买的机器是不是同一台？**
不是。测试 IP 是机房提供给你摸底用的节点，和具体套餐的宿主机不一定相同。它反映的是这条线路的路由质量，不是单台机器的性能。

**为什么我用 Pro 的测试 IP 测出来延迟很高？**
先确认你本地是哪个运营商。Pro 的三网都有优化，但如果本地宽带本身路由就不干净，测出来会偏。建议换手机热点（移动 / 联通各测一次）横向对比。

**Tier 1 到底能不能给国内用户用？**
如果你主要面向中国大陆用户，不建议。Tier 1 的定位就是国际业务，电信去程可能绕欧洲，延迟未必比美国节点低。它的价值在大带宽、大流量和超量后限速不断线。

**EB 标注 Beta 是什么意思？**
DMIT 官方目前仍将香港 Eyeball 标注为 Beta，产品与路由处于持续调整优化阶段，性能与路由都可能变动，官方也不建议用于对稳定性要求高的生产环境。买之前跑一次 LG，心里有个预期。

**流量跑完了会怎样？**
T1 系列超量后降速不停机（限速从 50Mbps 到 1Gbps 不等）。Pro 和 EB 系列的处理方式请以官网产品页说明为准，不建议按 T1 的规则去推。

## 测完再买，别反过来

香港节点最大的卖点是物理距离——离内地近，延迟天然比美西低一百多毫秒。但「香港」这两个字不保证任何事，真正决定体验的是回程路由。

所以顺序应该是：先用 103.117.100.199（Pro）、154.12.188.3（EB）、154.12.176.28（T1）这三个 IP 各 Ping 一轮，再去 `lg.dmit.sh` 跑回程 Traceroute，晚上八点再补测一次。三个数字摆在一起，该买哪条线路基本就清楚了。

如果你已经测完，确认移动或联通线路表现不错、预算又想压住，EB 系列是性价比最合理的落点；如果对延迟有硬要求并且能接受 $39.9/月起的价格，直接上 Pro；纯海外业务或者只需要一个香港 IP 做落地，T1 的 $36.9/年年付档几乎没什么可犹豫的。

[👉 前往 DMIT 查看当前香港套餐与库存](https://www.dmit.io/aff.php?aff=12046)

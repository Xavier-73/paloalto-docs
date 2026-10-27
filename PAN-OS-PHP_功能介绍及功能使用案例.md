# PAN-OS-PHP_功能介绍及功能使用案例

- Source: `PAN-OS-PHP_功能介绍及功能使用案例.pptx`
- Total slides: 16

## Slide 1

PALO ALTO NETWORKS 官方开源 · PAN-OS-PHP

防火墙配置自动化全景

PAN-OS-PHP 功能介绍

60+

100+

16

功能使用案例 · 脚本可复制 · 小白可照做

UTIL 类型

filter/action

迁移厂商

内部分享 · 以 fw-backup.xml 为主线 / 改动落新文件 / 干跑先行

UTIL 类型

filter/action

迁移厂商

terminal — 一句话批量

- $ pan-os-php type=rule in=/share/fw-backup.xml filter="(action is.allow)" actions=display
- → 瑞士军刀 · 60+ type · 100+ filter/action · fw-backup.xml 离线体检

★ 官方开源

▣ 瑞士军刀

⬢ 可落地

PaloAltoNetworks · 社区支持 AS IS

查改合清迁检 · 一箱全包

OrbStack + Docker · 离线零风险

github.com/pan-os-php

/lib + /utils + /migration

arm64 / amd64 全兼容

内部资料

内部分享 · 45 分钟 · 脚本均可复制 · 以 /share/fw-backup.xml 为主线

01 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 2

CONTENTS · 目录

目录 · Agenda

01

项目名片

是什么 / 架构 / 双模式

02′

✓ 会后可带走

02

环境部署

Docker CLI/API · 一键拉起

05′

- ● 60+ type 速查表 + 30+ 可复制脚本（落 fw-cleaned*.xml）
- ● /share/fw-backup.xml 主线：一键复现清理→体检→迁移
- ● ASA+Forti 双迁移底稿 + 回灌检查单
- ● 离线零风险 · P05 后简写 pan-os-php type=...

03

通用用法

type/in/out/filter/actions

03′

04

对象治理

地址·服务·标签·调度

07′

05

策略治理

规则编辑·合并·统计

07′

06

网络与系统

Zone/路由/接口/证书

04′

07

体检审计

xml-issue/size/diff/bpa

05′

08

跨厂商迁移

- 分享节奏 · 45 分钟
- 名片 2′ → 部署 5′ → 用法 3′ → 功能 30′（含迁移双页）→ 方法论 5′

ASA 深度 + Forti 双例

10′

09

管控与日志

上传·密钥·日志·API

04′

10

方法论

工作流·回灌·避坑

03′

02 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 3

01 PROFILE · 项目名片

PAN-OS-PHP 是什么

| 属性 | 说明 |
| --- | --- |
| 定位 | PAN-OS 配置的 PHP 对象化引擎，无需手写 XML |
| 维护 | PaloAltoNetworks 官方开源 · 社区支持 (AS IS) |
| 版本 | 2.1.x · PHP 8.x · d87de87 @ 2026-09 实测 |
| 依赖 | PHP + curl / dom / mbstring / bcmath / yaml (容器已内置) |
| 规模 | 617 地址 / 186 策略级生产库实测通过 |

★ 83

⑂ 34

Stars

Forks

✎ 369

▣ >30

Commits

UTIL

lib/pan_php_framework.php · composer.json · phpseclib

- ▣ 仓库结构速览
- ● /lib 核心库 · /utils 30+ CLI(pa_-edit/merger) · /migration 16 家厂商转换器 · /doc 类文档 · /tests 测试
- ● 一句话：防火墙运维的瑞士军刀 —— 查、改、合、清、迁、检，一个工具箱全包

03 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 4

02 ARCH · 架构总览

一次载入，三种用法

▦

⬢

◉

离线文件

对象引擎

在线 API

fw-backup.xml
cisco-asa-clean.txt

引用计数 / 过滤 / 合并

api://FW-IP · 命中数

零风险 · 只读写文件

PHP 8.x · 实测 617 级

读零风险 · 写走变更窗

CLI 主力

REST API

Web UI

filter + action · 一句话批量

JSON · 查为主 · 锁 8.1 镜像

点选拼命令 · 新人友好

- 核心心智 · 离线先行，线上只读 · /lib 对象化 + /utils 动作化 + /migration 转换化
- ● 本次全部功能离线可用 → 回灌才碰防火墙；API/UI 仅做命中数与查询辅助，不写生产。

04 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 5

03 SETUP · 环境部署

Docker 一键拉起 · 三种运行形态

● ● ● CLI 容器（推荐）— 本地离线

装 OrbStack/Docker

1

- 01
- 02
- 03
- 04

- docker pull swaschkut/pan-os-php:latest
- docker run --rm -v ${PWD}:/share -it swaschkut/pan-os-php:latest
- pan-os-php version
- pan-os-php type=stats in=/share/fw-backup.xml

dmg 拖拽 · 2s 启动 · 比 Desktop 轻一半

拉镜像

2

swaschkut/pan-os-php:latest · 原生 arm64/amd64

● ● ● API / Web UI 容器 — 在线辅助

- 01
- 02
- 03
- 04

- docker run -d -p 8082:80 swaschkut/pan-os-php-api:latest
- # UI: http://localhost:8082/utils/develop/ui
- # API: http://localhost:8082/utils/api/v1/tool.php/address?help
- # 等价 CLI: pan-os-php type=address help

拷配置

3

~/PANW/share ↔ /share 目录映射

- ▸ 依赖与本地构建
- ● PHP 8.x + curl / dom / mbstring / bcmath / yaml · 容器已内置
- ● 本地构建: docker build -f docker/Dockerfile-cli-ubuntu22 -t pan-os-php .
- ● 密钥管理: pan-os-php type=key-manager add=FW_IP user=admin pw=***

一句话跑

4

离线零风险 · API 按需连

05 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 6

04 USAGE · 通用用法

一套口径跑遍 60+ 工具

● ● ● 通用命令模板 — 记住这一行即可（P05 后简写）

- 01
- 02
- 03

- pan-os-php type=<type> in=<file|api://IP> [out=/share/fw-cleaned.xml] [location=any]
- [filter="(object is.unused) and (name contains prod)"] [actions=delete/display/export]
- [shadow-json / outputformatset / apitimeout=60]

| 参数 | 示例 | 说明 | 要点 |
| --- | --- | --- | --- |
| type | address / rule / zone | 60+ 类型，PH.php 定义 | listactions 看全量 |
| in | /share/fw-backup.xml | 离线文件（主线） | 离线零风险推荐 |
| filter | (object is.unused) | RQuery 引擎过滤 | 先 display 干跑 |
| actions | delete / display | 100+ 动作 (actions-*.php) | 可链式 a:b/c:d |
| location | any / vsys1 / DG | 作用域 | Panorama 用 DG |

- 三板斧：listactions → listfilters → display 干跑 → delete 落盘
- type=address listactions · type=rule listfilters · type=address actions=display

- 离线工作流本能（小白必读）
- in=/share/fw-backup.xml out=/share/fw-cleaned.xml 原件不动

06 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 7

05 OBJECTS · 对象治理

地址 · 服务 · 标签 · 调度 · 应用

| 类别 | type | 典型脚本 | 用途 |
| --- | --- | --- | --- |
| 地址/组 | address | filter=(object is.unused) actions=delete | 未使用清理 617→490 |
| 服务/组 | service | filter=(destport is 443) actions=delete | 端口协议治理 |
| 标签 | tag | actions=Color-set:blue / name-toLowerCase | 颜色/命名规范 |
| 调度 | schedule | filter=(name contains maint) actions=delete | 计划对象管理 |
| 应用 | application | filter=(name eq facebook) actions=display | 预定义/自定义应用 |
| 威胁 | threat | filter=(type is vulnerability) actions=display | 威胁对象 |
| Profile | securityprofile(group) | filter=(object is.unused) actions=delete | 安全Profile/组 |

● ● ● 脚本示例 — 对象治理（可复制·落新文件）

- 01
- 02
- 03
- 04
- 05
- 06

- # ① 先看再删（强烈推荐）
- pan-os-php type=address in=/share/fw-backup.xml 'filter=(object is.unused)' 'actions=display'
- pan-os-php type=address in=/share/fw-backup.xml out=/share/fw-cleaned.xml 'filter=(object is.unused)' 'actions=delete'
- # ② 服务/标签同口径
- pan-os-php type=service in=/share/fw-backup.xml out=/share/fw-cleaned.xml 'filter=(object is.unused)' 'actions=delete'
- pan-os-php type=tag in=/share/fw-cleaned.xml out=/share/fw-cleaned2.xml 'filter=(color is red)' 'actions=Color-set:blue'

💬 逐行解说：display 只看不改；out= 新文件（原件不动）；filter=(object is.unused) 找废物；Color-set 批量改属性

✓

▦

▣

−23

617→490

77→55

对象已清

地址

服务

fw-cleaned

含组展开

−28.6%

07 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 8

06 POLICY · 策略治理

规则编辑 · 合并 · 统计

| 类别 | type | 典型脚本 | 用途 |
| --- | --- | --- | --- |
| 规则编辑 | rule | filter=(action is.allow) actions=from-Add:dmz | 批量改源/目的/Zone/服务 |
| 规则合并 | rule-merger | in=fw.xml out=merged.xml location=any | 9维相似规则合并 |
| 对象合并 | address-merger | in=fw.xml out=merged.xml location=any | 地址/组减容 |
| …tag/service… | tag-merger/service-merger | 同上 | 标签/服务合并 |
| 统计 | stats | type=stats in=fw.xml | 对象/规则计数体检 |

● ● ● 脚本示例 — 策略治理（可复制）

- 01
- 02
- 03
- 04
- 05
- 06

- # ① 体检先行（只读）
- pan-os-php type=rule in=/share/fw-backup.xml 'filter=(action is.allow) and (service has any)' 'actions=display'
- pan-os-php type=stats in=/share/fw-backup.xml # 617 address · 186 policy · 49 nat · 64 pbf
- # ② 落盘：删禁用 + 合并（克制）
- pan-os-php type=rule in=/share/fw-backup.xml out=/share/fw-cleaned.xml 'filter=(action is.disabled)' 'actions=delete'
- pan-os-php type=rule-merger in=/share/fw-cleaned.xml out=/share/fw-cleaned2.xml location=any # 186→仅1对可合

💬 逐行解说：rule display 看放通；stats 报数；is.disabled 筛禁用落新文件；rule-merger 克制仅 1 对

- 实战：186 策略体检 · 0 完全重复 · 1 对可合 (203-1/203) · 5 全放通待收紧 · 168/168 日志全开
- filter 支撑：from has trust / service has any / action is.disabled / log-setting is.set

08 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 9

07 NETWORK · 网络与系统

Zone · 路由 · 接口 · 证书

| 类别 | type | 典型脚本 | 用途 |
| --- | --- | --- | --- |
| Zone | zone | type=zone in=api://FW actions=display | Zone 增删改 |
| 接口 | interface | type=interface in=fw.xml actions=display | 以太网/隧道/环回/聚合 |
| 路由 | routing/static-route | type=routing in=fw.xml actions=display | 虚拟路由/静态路由 |
| VirtualWire | virtualwire | type=virtualwire in=fw.xml actions=display | VWire |
| DHCP/证书 | dhcp / certificate | type=dhcp in=fw.xml actions=display | DHCP/证书管理 |
| 设备 | device / vsys | type=device in=panorama.xml actions=display | VSYS/DG/Template |

● ● ● 脚本示例 — 网络（只读为主）

- 01
- 02
- 03
- 04
- 05

- pan-os-php type=zone in=/share/fw-backup.xml actions=display # Zone 一览
- pan-os-php type=interface in=/share/fw-backup.xml actions=displayReferences # 接口及引用
- pan-os-php type=routing in=/share/fw-backup.xml actions=display # 虚拟路由
- pan-os-php type=static-route in=/share/fw-backup.xml 'filter=(destination is 0.0.0.0/0)' actions=display
- pan-os-php type=device in=/share/panorama.xml actions=display # Panorama DG/Template

💬 逐行解说：zone/interface/routing 只看不改；displayReferences 追引用；最后一行 Panorama 专用

说明：网络类以 display/审计为主，变更需谨慎；lib/network-classes/ 含 Zone/VR/IPsec/IKE

09 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 10

08 MERGER · 合并优化

减容 · 去重 · 缩编

地址合并

服务合并

标签合并

规则合并

address-merger

service-merger

tag-merger

rule-merger

−127 实测

−22 实测

去重增效

1 对可合

● ● ● 脚本示例 — 合并（离线 · 新文件）

- 01
- 02
- 03
- 04

- pan-os-php type=address-merger in=/share/fw-backup.xml out=/share/fw-merged.xml location=any
- pan-os-php type=addressgroup-merger in=/share/fw-backup.xml out=/share/fw-merged.xml location=any
- pan-os-php type=service-merger in=/share/fw-backup.xml out=/share/fw-merged.xml location=any
- pan-os-php type=rule-merger in=/share/fw-merged.xml out=/share/fw-merged2.xml location=any # 仅1对放行

💬 逐行解说：in=fw-backup 出 fw-merged 新文件；location=any 全域；rule-merger 最克制仅 1 对

- ▸ 使用要点（小白先看）
- ● 先 stats 看基数，再 merger 干跑（display/preview），确认后再 out 新文件
- ● 规则合并 9 维：26 条硬合曾被拒，仅放行最稳 1 对 —— 克制比激进更重要

10 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 11

09 AUDIT · 体检审计

完整性 · 体积 · 差异 · 最佳实践

| 类别 | type | 典型脚本 | 用途 |
| --- | --- | --- | --- |
| 完整性 | xml-issue | in=fw.xml out=fix.xml actions=fix | 几十项扫描：重名/空组/悬空 |
| 体积 | config-size | type=config-size in=fw.xml | 定位最占体积的配置段 |
| 差异 | diff | type=diff in1=a.xml in2=b.xml | 双配置 XML 差异 |
| 计数 | stats | type=stats in=fw.xml | 全量对象体检 |
| 运营 | xml-op-json | in=api://FW cmd="<show><system><info>" | 运维命令转 JSON |
| 最佳实践 | bpa-generator | type=bpa-generator in=api://FW | BPA 云打分（需 key） |

● ● ● 脚本示例 — 体检审计（门禁）

- 01
- 02
- 03
- 04
- 05

- pan-os-php type=xml-issue in=/share/fw-cleaned2.xml out=/share/fw-fix.xml actions=fix # 全绿才放行
- pan-os-php type=config-size in=/share/fw-cleaned2.xml # 634K 无膨胀
- pan-os-php type=stats in=/share/fw-cleaned2.xml # 617/186/49/64 体检
- pan-os-php type=diff in1=/share/fw-backup.xml in2=/share/fw-cleaned2.xml # diff 留痕
- pan-os-php type=bpa-generator in=api://FW # 需 bpa@ key

💬 逐行解说：xml-issue 全绿门禁；config-size 看体积；stats 数数；diff 对比新旧；bpa 需 key

- 门禁实测：fw-cleaned2 / cisco-migrated2 均为 0 修复项 · 168/168 日志全开
- 铁律：xml-issue 全绿是放行硬门槛，非绿不进窗

11 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 12

10 MIGRATION-A · 跨厂商迁移（1/2）

Cisco ASA 9.20 · 1949 行 → PAN-OS 可用底稿

| 配置项 | 源 ASA | 转换结果 | 转换率 | 状态 |
| --- | --- | --- | --- | --- |
| 地址/服务 | 73 组+对象 | 498 / 136 | — | ✓ 0 报错 |
| 安全策略 | 39 ACL | 36 条 | 约 92% | ✓ 归并 |
| NAT | Twice+Auto | 17 条 | 100% | ✓ |
| Zone/接口/路由 | — | 4 / 8 / 7 条 | — | ✓ |
| VPN | 50 行 AnyConnect | 0 | 0% | ✕ GP 重建 |

● ● ● 可复制脚本 — ASA 迁移全链路（落新文件）

- 01
- 02
- 03
- 04
- 05
- 06
- 07
- 08

- # ① 准备空防火墙底座（/share 已放 firewall_base.xml）
- pan-os-php type=vendor-migration in=/share/firewall_base.xml \
- out=/share/cisco-migrated.xml file=/share/cisco-asa-clean.txt vendor=ciscoasa
- # ② 体检 + 校验（0 报错才算底稿）
- pan-os-php type=stats in=/share/cisco-migrated.xml
- pan-os-php type=xml-issue in=/share/cisco-migrated.xml # 全绿放行
- # ③ 迁移后再清理（复用 fw 套路）
- pan-os-php type=address in=/share/cisco-migrated.xml out=/share/cisco-migrated2.xml 'filter=(object is.unused)' 'actions=delete'

💬 逐行解说：in= 底座；file= ASA 源；vendor= ciscoasa；out= 新配置；stats/xml-issue 全绿才入库

✓ 可照抄：cisco-asa-clean.txt + firewall_base.xml → cisco-migrated.xml → 校验 → 清理 → cisco-migrated2.xml

12 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 13

10 MIGRATION-B · 跨厂商迁移（2/2）

Cisco ASA 进阶 + 16 家厂商一键转

| 厂商 | 型号 | vendor 参数 | 状态 |
| --- | --- | --- | --- |
| Cisco | ASA | ciscoasa | ✓ 实测 90% |
| Cisco | ISR/Switch | ciscoisr / ciscoswitch | ✓ |
| Check Point | <R80 / ≥R80 | cp / cp-r80 | ✓ |
| Juniper | SRX / ScreenOS | srx / netscreen | ✓ |
| Fortinet等 | Forti/Sonic/Sophos | fortinet / sonicwall / sophos | ✓ |
| Huawei等 | Huawei/Stonesoft | huawei / stonesoft | ✓ |

● ● ● 可复制脚本 — ASA 进阶 / 校验 / SSH 拉取（参考原稿口径）

- 01
- 02
- 03
- 04
- 05
- 06
- 07
- 08

- # ASA 底座用 firewall_base.xml（原稿 panos_base.xml 已修正）
- pan-os-php type=vendor-migration in=/share/firewall_base.xml \
- out=/share/cisco-migrated.xml file=/share/cisco-asa-clean.txt vendor=ciscoasa
- # SSH 拉取 + 就地转（适合现网，原稿 ssh-connector）
- pan-os-php type=ssh-connector host=10.0.0.1 user=admin # 先抓 live 配置到 file
- # 统一校验收口
- pan-os-php type=xml-issue in=/share/cisco-migrated.xml out=/share/cisco-fix.xml actions=fix
- pan-os-php type=stats in=/share/cisco-migrated.xml # 0 报错 + 规模同 12 页

💬 逐行解说：ssh-connector 抓 live 到 file；vendor-migration 选 ciscoasa 一键转；xml-issue/stats 校验

缺口需人工：AnyConnect 50 行 → GP 重建 · tunneled 路由 metric 100 隔离 · 接口重挂

13 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 14

11 OPS · 管控与日志

上传 · 密钥 · 日志 · 自动化

| 类别 | type | 典型脚本 | 用途 |
| --- | --- | --- | --- |
| 导入导出 | upload | in=/share/fw-cleaned2.xml out=api://FW | 配置上传/下载 |
| 密钥 | key-manager | add=FW user=admin pw=*** / list | .panconfkeystore 管理 |
| 覆盖 | override-finder | in=api://FW | Panorama 模板覆盖定位 |
| AppID | appid-enabler | in=api://FW actions=enable | 禁用 AppID 一键启用 |
| IP/用户 | register-ip-mgr / userid-mgr | in=api://FW actions=register | IP/用户映射 |
| 日志 | traffic-log / system-log | in=api://FW filter="(app is facebook)" | 流量/系统日志过滤 |
| 系统 | software-* / license / ike | in=api://FW | 软件/许可/IKE 展示 |

● ● ● 脚本示例 — 管控（落新文件→上 candidate）

- 01
- 02
- 03
- 04
- 05
- 06
- 07

- # 配置管控（/share 主线）
- pan-os-php type=upload in=/share/fw-cleaned2.xml out=api://FW # 导入 candidate
- pan-os-php type=key-manager add=10.0.0.1 user=admin pw=*** # 存 key
- pan-os-php type=diff in1=/share/fw-backup.xml in2=/share/fw-cleaned2.xml # 差异留痕
- # 日志与辅助
- pan-os-php type=traffic-log in=api://FW filter="(app is facebook)" shadow-json
- pan-os-php type=ike in=/share/fw-backup.xml # IKE/IPsec 一览

💬 逐行解说：upload 推新文件到 candidate；key-manager 管密钥；diff 留痕；traffic-log shadow-json

提示：upload / commit 走变更窗口；ike/系统类以 display 为主，变更前先看

14 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 15

13 METHOD · 方法论

工作流 · 回灌 · 避坑

STEP 1

STEP 2

STEP 3

STEP 4

STEP 5

备份

清理

体检

迁移

回灌

running-config
+ ASA 文本

无用/禁用
重复

策略/NAT/PBF
完整性

ASA→PAN-OS
底稿

Import·Validate
·Commit

- 回灌检查单 · Import → Validate → Commit
- ● ① 再导 running-config 双保险 ② upload 进 candidate（整 file/xpath）
- ● ③ GUI Validate warning 逐条过 ④ 窗口内 Commit 看进度+自动回滚 ⑤ 抽查 186/41/53 + 业务抽测

- ▸ 避坑 Top 3
- ● Zoom orphan：组删后第二遍才现形 → 必二次 display
- ● NAT 误报：TCP/UDP 逐字节对，勿跨协议合
- ● API 挑版本：锁 pan-os-php-api:8.1

铁律：机械量大归工具，要判断归人 · 凡删改必看原始字段、留回滚链

| 门禁 | 标准 | 实测 | 结论 |
| --- | --- | --- | --- |
| xml-issue | 0 手工修复 | fw-cleaned2 全绿 | ✓ 放行 |
| config-size | 无膨胀 | 634K 小配置 | ✓ |
| 日志覆盖 | allow 全开 | 168/168 | ✓ |

15 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

## Slide 16

谢谢

Q & A · 现场演示：type=stats 干跑 10 秒出体检

- 项目：github.com/PaloAltoNetworks/pan-os-php · 文档：pan-os-php/doc/index.html
- 产出：/share/fw-cleaned2.xml · 容器：swaschkut/pan-os-php:latest · 密钥：key-manager
- 口令：离线先行 · 干跑先行 · 全绿放行

16 / 16

PAN-OS-PHP · 防火墙配置自动化 · 内部分享

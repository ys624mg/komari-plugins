# Komari 插件源（个人）

> 管理源 JSON：`https://raw.githubusercontent.com/ys624mg/komari-plugins/main/v1.json`

## 在 Komari 中添加本插件源

1. 打开 Komari 管理后台 → **插件市场** → **管理源 / 添加源**。
2. 源名称随意（如 `ys624mg-plugins`）。
3. 源地址粘贴上面的管理源 JSON 链接。
4. 保存后即可在插件市场看到下方插件。

## 插件列表

| 插件 short | 名称 | 最新版本 | SHA256 | 下载 |
| --- | --- | --- | --- | --- |
| `resource-alert` | 资源监控报警 | 1.12.0 | `03910f47f8995ebf…` | [下载](https://github.com/ys624mg/komari-plugins/releases/download/resource-alert-1.12.0/resource-alert-1.12.0.zip) |
| `narwhal-finance` | 独角鲸托管 | 2.2.0 | `5fd2447a7ceedd27…` | [下载](https://github.com/ys624mg/komari-plugins/releases/download/narwhal-finance-2.2.0/narwhal-finance-2.2.0.zip) |
| `month-rotate` | 流量月重置日管理 | 1.5.0 | `2af38c0d177029d2…` | [下载](https://github.com/ys624mg/komari-plugins/releases/download/month-rotate-1.5.0/month-rotate-fixed-1.5.0.zip) |
| `acck-checkin` | ACCK 自动签到 | 3.9.5 | `d2a5988dcb38c559…` | [下载](https://github.com/ys624mg/komari-plugins/releases/download/acck-checkin-3.9.5/acck-checkin-3.9.5.zip) |

## 插件说明

### 资源监控报警 (`resource-alert`)

- 版本：**1.12.0**
- 作者：Komari-User
- Komari 要求：`>=0.0.1`
- 下载：https://github.com/ys624mg/komari-plugins/releases/download/resource-alert-1.12.0/resource-alert-1.12.0.zip
- SHA256：`03910f47f8995ebf62b27bc40fc2ec31ab34fdc999d8f6b0e6ec11c3197ef9a1`
- 说明：定时检测所有节点 CPU 使用率、系统负载与连接数，超过阈值时通过 Komari 通知渠道或自定义 Webhook 报警，恢复后发送恢复消息。另支持「IP 被墙检测」：引用 Komari 自带延迟监测任务的 ping 数据，选中的任务持续超时达到设定时长即判定入站 IP 被封锁并通过站点自带通知渠道报警，恢复后自动通知。

### 独角鲸托管 (`narwhal-finance`)

- 版本：**2.2.0**
- 作者：Komari-User
- Komari 要求：`>=0.0.1`
- 下载：https://github.com/ys624mg/komari-plugins/releases/download/narwhal-finance-2.2.0/narwhal-finance-2.2.0.zip
- SHA256：`5fd2447a7ceedd273aeb6314e29fcbe002195a6d5b1ee38f8b7fbba022cb78a2`
- 说明：独角鲸托管（Narwhal Cloud）一体化看板：汇总托管实例、预计月入、累计提现与四类余额变化，提供机器概览、VM 归档、客户排行、退款统计与日报投递记录。支持 Telegram 日报、新交易与工单推送、自动提现，并可在面板内管理套餐、促销码与机器信息。

### 流量月重置日管理 (`month-rotate`)

- 版本：**1.5.0**
- 作者：Komari-User
- Komari 要求：`>=1.5.0`
- 下载：https://github.com/ys624mg/komari-plugins/releases/download/month-rotate-1.5.0/month-rotate-fixed-1.5.0.zip
- SHA256：`2af38c0d177029d2e6d246c789852216f5959d6d1599b036177868d27ceaceae`
- 说明：集中查看和批量修改每个节点 Komari Agent 的「网络统计月重置日」（--month-rotate 参数）与面板节点的「流量阈值/统计方式」（traffic_limit / traffic_limit_type）。自动识别节点环境（systemd 服务/用户级服务/容器/手动运行）；手动运行的 Agent 可一键迁移为 systemd 管理（自动创建服务、接管进程并设置重置日）。流量阈值为面板原生字段即时生效，单位默认 GB，按 Komari 1024 进制自动换算，统计方式默认总和。

### ACCK 自动签到 (`acck-checkin`)

- 版本：**3.9.5**
- 作者：Komari-User
- Komari 要求：`>=0.0.1`
- 下载：https://github.com/ys624mg/komari-plugins/releases/download/acck-checkin-3.9.5/acck-checkin-3.9.5.zip
- SHA256：`d2a5988dcb38c559eb19f98cadb2a89774b7c993ed0d89ed17700163deb882e5`
- 说明：支持多个 ACCK (acck.io) 账号自动登录并完成每日签到：逐个求解 altcha 工作量证明，用邮箱+密码+2FA(TOTP) 换取 Token，定时批量签到领取 AC 币；签到完成后统一推送一条「批量签到汇总」，不逐账号骚扰。另支持 AC 交易区监控：监控只负责发现新上架并推送——推送范围可选「全部新上架」或「仅命中规则的」，「推送规则」与「自动交易」两套规则各自独立；推送消息附带 TG 内联购买按钮，可开启二次确认防误点。「自动交易」按交易规则列表匹配后自动下单，账号、每轮上限与被墙过滤单独控制。敏感数据保护：密码与 TOTP 密钥不回显，邮箱一律脱敏展示。配置页改动（含开启交易所监控）即时生效，无需重启插件。

v3.9.0：管理页全面重做（分页签 + KPI + 实时进度 + 购买弹窗）；修复「购买失败后无法重试」「签到记录被固定截断为 20 条」「手动签到并发未拦截」等问题；PoW 求解增加超时保护；状态文件改为原子写入并合并落盘；规则条件支持「价格」别名与全角数字。

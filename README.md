# 我的 Komari 插件源（管理源）

按官方 [komari-monitor/plugin-market](https://github.com/komari-monitor/plugin-market) 的 `v1.json` 格式维护的第三方 Komari 插件源，插件包通过 GitHub Releases 分发。

## 管理源 JSON 链接

在 Komari 后台「插件市场 → 管理源」中添加以下地址即可订阅本源：

```
https://raw.githubusercontent.com/ys624mg/komari-plugins/main/v1.json
```

## 使用方式

1. 登录 Komari 面板 → 左侧 **插件市场** → 右上角齿轮图标（管理源）。
2. 点击「添加源」：名称随意（例如「我的插件源」），URL 填上面的链接，保存并刷新。
3. 返回插件市场即可看到下方插件，直接安装 / 更新（带更新时会显示「Update available」）。

## 插件列表

| 插件 | 版本 | Komari 要求 | 下载 |
| --- | --- | --- | --- |
| ACCK 自动签到 (acck-checkin) | 3.9.0 | Komari >=0.0.1 | [resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/acck-checkin-3.9.0/acck-checkin-3.9.0.zip) || 流量月重置日管理 (month-rotate) | 1.5.0 | Komari >=1.5.0 | [resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/month-rotate-1.5.0/month-rotate-fixed-1.5.0.zip) || 独角鲸托管财务日报 (narwhal-finance) | 1.11.1 | Komari >=0.0.1 | [resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/narwhal-finance-1.11.1/narwhal-finance-1.11.1.zip) || 资源监控报警 (resource-alert) | 1.12.0 | Komari >=0.0.1 | [resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/resource-alert-1.12.0/resource-alert-1.12.0.zip) |

## 插件详情

### ACCK 自动签到 / ACCK Auto Check-in （`acck-checkin`） v3.9.0
- 要求：Komari >=0.0.1
- 下载：[resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/acck-checkin-3.9.0/acck-checkin-3.9.0.zip)
- SHA256：`54b19ac0cb34109f9a649285a1b165c231d6545ca673e9f00bc3dd03b3c13b3c`

支持多个 ACCK (acck.io) 账号自动登录并完成每日签到：逐个求解 altcha 工作量证明，用邮箱+密码+2FA(TOTP) 换取 Token，定时批量签到领取 AC 币；签到完成后统一推送一条「批量签到汇总」，不逐账号骚扰。另支持 AC 交易区监控：监控只负责发现新上架并推送——推送范围可选「全部新上架」或「仅命中规则的」，「推送规则」与「自动交易」两套规则各自独立；推送消息附带 TG 内联购买按钮，可开启二次确认防误点。「自动交易」按交易规则列表匹配后自动下单，账号、每轮上限与被墙过滤单独控制。敏感数据保护：密码与 TOTP 密钥不回显，邮箱一律脱敏展示。配置页改动（含开启交易所监控）即时生效，无需重启插件。

v3.9.0：管理页全面重做（分页签 + KPI + 实时进度 + 购买弹窗）；修复「购买失败后无法重试」「签到记录被固定截断为 20 条」「手动签到并发未拦截」等问题；PoW 求解增加超时保护；状态文件改为原子写入并合并落盘；规则条件支持「价格」别名与全角数字。
### 流量月重置日管理 / Month Rotate Manager （`month-rotate`） v1.5.0
- 要求：Komari >=1.5.0
- 下载：[resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/month-rotate-1.5.0/month-rotate-fixed-1.5.0.zip)
- SHA256：`2af38c0d177029d2e6d246c789852216f5959d6d1599b036177868d27ceaceae`

集中查看和批量修改每个节点 Komari Agent 的「网络统计月重置日」（--month-rotate 参数）与面板节点的「流量阈值/统计方式」（traffic_limit / traffic_limit_type）。自动识别节点环境（systemd 服务/用户级服务/容器/手动运行）；手动运行的 Agent 可一键迁移为 systemd 管理（自动创建服务、接管进程并设置重置日）。流量阈值为面板原生字段即时生效，单位默认 GB，按 Komari 1024 进制自动换算，统计方式默认总和。
### 独角鲸托管财务日报 / Hosting Finance Daily （`narwhal-finance`） v1.11.1
- 要求：Komari >=0.0.1
- 下载：[resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/narwhal-finance-1.11.1/narwhal-finance-1.11.1.zip)
- SHA256：`e0b47e31dbc71c2669815894642542b065a42bf1c98cac2a363b94ed9e0345f7`

通过 Narwhal Cloud(fuckip.me) 开放 API 拉取托管财务数据：托管实例数、预计月入、累计提现、托管/账户/冻结/可用余额变化、今日入账与扣减汇总、当日交易明细。每日定时通过自定义 Telegram Bot 发送日报，也可开启新交易实时推送；管理页提供可视化看板（余额对比、收支构成、交易流水、客户排行、机器套餐库存、VM 归档、退款统计、日报预览与投递记录）。
### 资源监控报警 / Resource Alert Monitor （`resource-alert`） v1.12.0
- 要求：Komari >=0.0.1
- 下载：[resource-alert-1.12.0.zip](https://github.com/ys624mg/komari-plugins/releases/download/resource-alert-1.12.0/resource-alert-1.12.0.zip)
- SHA256：`03910f47f8995ebf62b27bc40fc2ec31ab34fdc999d8f6b0e6ec11c3197ef9a1`

定时检测所有节点 CPU 使用率、系统负载与连接数，超过阈值时通过 Komari 通知渠道或自定义 Webhook 报警，恢复后发送恢复消息。另支持「IP 被墙检测」：引用 Komari 自带延迟监测任务的 ping 数据，选中的任务持续超时达到设定时长即判定入站 IP 被封锁并通过站点自带通知渠道报警，恢复后自动通知。


---
本仓库仅维护 `v1.json` 与说明；插件包以 GitHub Releases 资产形式提供。

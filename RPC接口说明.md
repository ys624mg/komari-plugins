# 独角鲸托管 · RPC 对外接口说明

适用版本：**2.19.22**　核对日期：2026-11-08　插件 short：`narwhal-finance`

这份文档描述本插件向**其它 Komari 插件**开放的全部 RPC 方法（通过 `server.call` 调用），以及与之共用同一套 action 的 HTTP 通道。

---

## 一、先打开开关

所有对外能力都在「插件设置 → ⑨ 插件间联动」下，默认值是保守的：

| 配置项 | 作用 | 默认 |
|---|---|---|
| 允许其他插件调用 | 总开关，关掉后所有 RPC 一律返回 `disabled` | **开** |
| 对外调用令牌（选填） | 留空 = 不校验；填了就要求每次带 `token` | 空 |
| 允许 HTTP 接口调用 | 只影响 HTTP 通道，不影响 RPC | **关** |
| HTTP 每分钟调用上限 | 只对 HTTP 生效，RPC 不限速 | 120 |
| 允许接口通知买家 | `notify.buyer` 的总闸，关着时该 action 返回 `notify_disabled` | **关** |
| 通知冷却秒数 | 同一 VM + 同一内容在该窗口内只发一次，0 = 不限 | 60 |

准入顺序：**总开关 → 令牌**。RPC 侧不做限速（宿主本身是进程内调用，不需要）。

---

## 二、调用约定

```js
// 在其它插件的 script.js 里（load() 中即可，不需要登录态）
const r = await server.call("plugin:narwhalFinanceApi", {
  action: "vm.lookup",
  vm: "8ae6c68e-1f2a-4c3b-9d7e-5a6b7c8d9e0f",
  token: "<令牌，配置里留空则不用带>",
});
```

**同步约束（必读）**

Komari 的 RPC handler 是**同步调用**的，返回 Promise 不会被等待。本插件所有 handler 都已按同步实现，`await` 只是为了写法统一——你拿到的永远是最终对象，不会是 `undefined`。

反过来：如果你自己的插件把 handler 写成 `async`，**调用方只会拿到空壳**，那是调用方插件的问题，不是本插件的。

**令牌传参**：`token` 也可以用 `api_token` / `key` / `access_token` 任一别名。HTTP 通道额外支持 `X-Api-Token` / `X-Narwhal-Token` 请求头。

---

## 三、方法总览（共 8 个）

| RPC 方法 | 用途 | 过开关+令牌 | 联网 |
|---|---|---|---|
| `plugin:narwhalFinanceApi` | 通用 action 分发，覆盖 8 个 action | 是 | 视 action |
| `plugin:narwhalFinanceVmLookup` | 按容器 ID 反查买家归属 | 是 | 否 |
| `plugin:narwhalFinanceTask` | 查询后台任务终态 | 是 | 否 |
| `plugin:narwhalFinancePoll` | 触发一轮数据轮询 | 否 | 是（后台） |
| `plugin:narwhalFinanceReport` | 触发一次日报生成并推送 | 否 | 是（后台） |
| `plugin:narwhalFinanceTgPoll` | 触发一次 Telegram 更新拉取 | 否 | 是（后台） |
| `plugin:narwhalFinanceRank` | 触发一次客户排行重算 | 否 | 是（后台） |
| `plugin:narwhalFinanceRenewRemind` | 触发一次续费提醒 | 否 | 是（后台） |

后 5 个是**内部定时任务的触发器**，给外部插件补跑用，不经过对外接口开关，也不需要令牌。

---

## 四、action 详解

### 4.1 `ping` — 连通性测试

无参数。

```json
{ "ok": true, "action": "ping", "version": "2.19.22", "time": 1762300000, "serverTime": "2026-11-05T10:00:00.000Z" }
```

---

### 4.2 `vm.lookup` — 按容器 ID 反查买家

| 参数 | 说明 |
|---|---|
| `vm` | 容器 ID。也接受 `container` / `containerId` / `prefix` / `id` |

**输入限制**：只接受 36 位 UUID，或 8 位以上十六进制前缀。**不支持按 IP 反查**（IP 会回收再分配，容易查到别的买家），也不认自定义容器名（可能重复）。

```json
{
  "ok": true, "action": "vm.lookup", "found": true,
  "vm": {
    "vmId": "8ae6c68e-1f2a-4c3b-9d7e-5a6b7c8d9e0f",
    "containerId": "8ae6c68e-1f2a-4c3b-9d7e-5a6b7c8d9e0f",
    "shortId": "8ae6c68e",
    "ip": "10.0.0.12",
    "email": "buyer@example.com",
    "userName": "buyer",
    "buyerName": "buyer",
    "machineId": "m_xxx", "machineName": "香港-01", "machineRegion": "HK",
    "planId": "p_xxx", "planName": "标准型",
    "cpu": 2, "ramMb": 2048, "diskGb": 20,
    "trafficGb": 500, "trafficUsedGb": 123.45, "bandwidthMbps": 100,
    "price": 29.9, "status": "running", "state": "active",
    "createdAt": 1750000000, "expiresAt": 1762300000, "daysLeft": 30,
    "autoRenew": true, "archived": false, "source": "narwhal-finance"
  }
}
```

未收录时返回 `{ "ok": true, "action": "vm.lookup", "found": false, "vm": null, "error": "未收录该容器（可能已销毁且不在归档中）" }`。
参数格式错误时 `ok` 为 `false`。

> 字段以 `externalVmRow()` 归一化后的结果为准，与 `vm.list` 的条目完全一致。

---

### 4.3 `vm.list` — 批量列实例

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `expiringDays` | number | 不过滤 | 只看 N 天内到期（0 = 今天到期）。永久实例不参与 |
| `email` | string | — | 按买家邮箱精确过滤（不区分大小写） |
| `machineId` | string | — | 按母鸡过滤 |
| `includeArchived` | bool | `false` | 是否带上归档实例 |
| `includeExpired` | bool | `false` | 配合 `expiringDays`：是否包含已过期的 |
| `limit` | number | 500 | 1–2000 |

```json
{ "ok": true, "action": "vm.list", "total": 42, "count": 42, "limit": 500,
  "items": [ { "vmId": "...", "...": "字段与 vm.lookup 的 vm 完全一致" } ] }
```

`total` 是过滤后的总数，`count` 是本次实际返回条数（受 `limit` 限制）。结果按到期时间升序，永久实例排在最后。

---

### 4.4 `machine.list` — 母鸡清单

无参数。

```json
{
  "ok": true, "action": "machine.list", "total": 3, "count": 3,
  "items": [{
    "id": "m_xxx", "name": "香港-01", "region": "HK", "status": "active",
    "description": "", "tags": ["hk"], "entryHost": "hk-01.example.cloud",
    "acckTrafficGb": 1200.5,
    "isActive": true, "expiresAt": 1762300000,
    "estRevenue": 899.7, "earnedRevenue": 5600.2,
    "specs": { "cpus": 32, "ramTotalMb": 65536, "diskTotalGb": 2000, "bandwidthMbps": 1000 },
    "planCount": 4, "vmCount": 42, "activePlans": 4,
    "source": "narwhal-finance"
  }]
}
```

`acckTrafficGb` 与 `earnedRevenue` **可能为 `null`**（未配 ACCK Token / 平台没给该字段），调用方要据此区分「没数据」和「真的是 0」。

---

### 4.5 `plan.list` — 套餐清单

| 参数 | 说明 |
|---|---|
| `machineId` | 可选，只列该母鸡下的套餐 |

```json
{
  "ok": true, "action": "plan.list", "total": 4, "count": 4,
  "items": [{
    "machineId": "m_xxx", "machineName": "香港-01", "machineRegion": "HK",
    "planId": "p_xxx", "name": "标准型", "description": "",
    "price": 29.9, "cpu": 2, "ramMb": 2048, "diskGb": 20,
    "trafficGb": 500, "bandwidthMbps": 100,
    "limitPerUser": false, "allowFullRefund": true,
    "max": 50, "sold": 12, "remaining": 38, "unlimited": false,
    "status": "active", "vms": 12, "source": "narwhal-finance"
  }]
}
```

`remaining` 未知时为 `-1`。

---

### 4.6 `stats.summary` — 汇总统计

无参数。

```json
{
  "ok": true, "action": "stats.summary",
  "summary": {
    "version": "2.19.22",
    "generatedAt": "2026-11-05T10:00:00.000Z",
    "instanceCount": 42,
    "estMonthly": 1256.4,
    "currency": "¥",
    "machines": 3, "plans": 9, "vms": 42,
    "expiringIn7Days": 5,
    "archive": { "total": 128, "live": 120, "refunded": 6, "expired": 2 },
    "source": "narwhal-finance"
  }
}
```

`generatedAt` 取自最近一次机器同步时间，`null` 表示还没同步过。

两个字段的口径（2.19.20 起）：

- `expiringIn7Days` 只数**尚未过期**且 7 天内到期的实例，与 `vm.list` 的
  `expiringDays=7`（默认 `includeExpired=false`）同一口径。已过期、但还没从平台
  VM 列表里消失的实例**不计入**。
- `instanceCount` 是各母鸡 `vm_count` 之和；平台明确返回 `0` 时就是 `0`。
  只有**取不到**该字段（老接口 / 精简缓存）才回落到母鸡台数。

---

### 4.7 `notify.buyer` — 给买家发站内通知（写操作）

| 参数 | 说明 |
|---|---|
| `vm` | 容器 ID / 8 位前缀（推荐）。也可用 `container` / `containerId` / `prefix` |
| `machineId` + `userId` | 不用 `vm` 时的替代定位方式；`userId` 可换 `email` 反查 |
| `subject` | 通知标题，≤ 200 字，必填 |
| `message` | 通知正文，≤ 2000 字，必填（别名 `content` / `body`） |
| `dryRun` | `true` 时只解析目标不发送（别名 `dry_run`） |

需要先在设置里打开「允许接口通知买家」。平台限制每台 VM 每 24 小时最多 10 条，超了平台报错会原样透出。

**RPC 侧是两段式**：能同步判定的（校验 / 解析目标 / dryRun / 冷却）直接返回结果；真要联网发送时登记后台任务并**立即返回 taskId**。

dryRun：

```json
{
  "ok": true, "action": "notify.buyer", "dryRun": true, "notified": 0,
  "target": { "vmId": "...", "shortId": "8ae6c68e", "email": "...", "userId": "...",
              "machineId": "...", "machineName": "...", "planName": "", "archived": false },
  "subject": "流量告警", "messageLength": 42,
  "message": "dryRun：已解析到买家，未实际发送"
}
```

冷却命中：同上结构，但 `duplicate: true`、`message` 说明冷却剩余窗口。

真正发送（入队）：

```json
{
  "ok": true, "action": "notify.buyer", "queued": true,
  "taskId": "nfm9x2k1", "state": "running",
  "target": { "...": "同上" }, "subject": "流量告警",
  "note": "RPC 为同步取值，联网动作已转后台；用 plugin:narwhalFinanceTask 或 action=task.get 取结果"
}
```

---

### 4.8 `task.get` — 查后台任务终态

| 参数 | 说明 |
|---|---|
| `taskId` | 上一步拿到的 `taskId`（别名 `task_id` / `id`） |

```json
{
  "ok": true, "action": "task.get", "taskId": "nfm9x2k1",
  "state": "done", "running": false,
  "createdAt": 1762300000, "finishedAt": 1762300003,
  "result": { "ok": true, "action": "notify.buyer", "notified": 1, "target": { "...": "" } }
}
```

`state` 取值：`running` / `done` / `failed` / `unknown`。
任务记录保留 **1 小时**，最多 100 条，过期返回 `{ "ok": false, "state": "unknown", "error": "任务不存在或已过期（保留 1 小时）" }`。

---

## 五、`plugin:narwhalFinanceVmLookup`

给资源告警这类插件做归属反查，比走 `Api` 少一层 action 解析。

```js
const r = await server.call("plugin:narwhalFinanceVmLookup", { container: "8ae6c68e" });
if (r.ok && r.found) console.log(r.vm.email, r.vm.machineName);
```

| 参数 | 说明 |
|---|---|
| `container` | 容器 ID / 8 位前缀（别名 `vm` / `prefix` / `id`） |

返回 `{ ok: true, found: true, vm }`，或 `{ ok: true, found: false, vm: null, error: "财务插件未收录该容器" }`，或 `{ ok: false, found: false, error }`。

> **与 `vm.lookup` action 的差异**：这里的 `vm` 是 `lookupContainerOwner()` 的原始对象，**没有**经过 `externalVmRow()` 归一化——**没有 `daysLeft` 和 `buyerName`**，但多出 `lastSeenAt` 和 `refunded`。要统一字段请走 `Api` 的 `vm.lookup`。

---

## 六、触发型 RPC（5 个）

```js
await server.call("plugin:narwhalFinancePoll", {});       // 数据轮询
await server.call("plugin:narwhalFinanceReport", {});     // 日报
await server.call("plugin:narwhalFinanceTgPoll", {});     // TG 更新拉取
await server.call("plugin:narwhalFinanceRank", {});       // 客户排行重算
await server.call("plugin:narwhalFinanceRenewRemind", {});// 续费提醒
```

统一返回 `{ ok: true, note: "... scheduled in background" }`。

**行为边界（重要）**：这几个都是「登记后立即返回」，**不保证真的执行**。任务本身有并发保护，上一轮还在跑时这次调用会被静默跳过，返回体仍然是 `ok: true`。需要确认真跑了，去管理页看「最近同步 / 日报投递记录」，不要只信返回值。

`RenewRemind` 是唯一会先判配置的：续费提醒没启用时返回 `{ "ok": false, "note": "续费提醒未启用，请先在插件设置里打开" }`。

---

## 七、错误对象

RPC 失败统一返回 `{ ok: false, error, message? }`，**不抛异常**：

| `error` | 含义 |
|---|---|
| `disabled` | 「允许其他插件调用」已关闭 |
| `unauthorized` | 令牌不匹配 |
| `notify_disabled` | 「允许接口通知买家」未开启 |
| `unknown`（task） | 任务不存在或已过期 |
| 其它中文串 | 参数问题 / 业务失败，`error` 里直接是原因 |

未知 action：

```json
{ "ok": false, "action": "xxx", "error": "未知 action：xxx（支持 ping / vm.lookup / vm.list / machine.list / plan.list / stats.summary / notify.buyer / task.get）" }
```

---

## 八、HTTP 通道（与 RPC 共用 action）

开关独立，需要额外打开「允许 HTTP 接口调用」。

```
GET  /narwhal-finance/open/api?action=vm.lookup&vm=<容器ID>&token=<令牌>
POST /narwhal-finance/open/api        {"action":"notify.buyer","vm":"...","subject":"...","message":"..."}
```

请求头也认令牌：`X-Api-Token: <令牌>` 或 `X-Narwhal-Token: <令牌>`。

| 状态码 | 含义 |
|---|---|
| 200 | 成功（业务失败也是 200，看 `ok` 字段） |
| 401 | 令牌错误 |
| 403 | 总开关关闭 / HTTP 未开启 |
| 429 | 超过每分钟调用上限 |
| 422 | 服务端异常（**不用 5xx**：面板网关会把 5xx 当插件崩溃并替换响应体） |

**与 RPC 的唯一行为差异**：HTTP 是异步通道，`notify.buyer` 会**直接等发送结果**返回终态，不会给你 `taskId`。需要两段式就调 RPC。

---

## 九、版本与变更

| 版本 | RPC 相关变更 |
|---|---|
| 2.19.22 | **无变更**。本版只改派生缓存的失效判据：`machineDataStamp()` 加了「内容版本」维度，单机重同步 / 改机器名 / 改套餐名之后 `/status` 的交易摘要行（`txns`）立刻反映新名字，不再等到下一次全量同步；改套餐名时归档里的 `planName` 一并回写。方法名 / 参数 / 返回字段 / 取值口径都没动 |
| 2.19.21 | **无变更**。本版只改管理页与日报里的提现口径：`提现失败 / 已撤销` 不再计入「今日提现」与收入明细的「提现」列（此前会算进去，导致「累计提现」没动而「今日提现」虚高），也不记成扣减。对外接口不返回提现金额，方法名 / 参数 / 字段名都没动 |
| 2.19.20 | **行为修正，契约不变**。`stats.summary` 的 `expiringIn7Days` 不再包含已过期实例（与 `vm.list` 的 `expiringDays` 同口径，此前把已过期但还没从 VM 列表消失的实例也算进「7 天内到期」）；`stats.summary` 的 `instanceCount` 在平台明确返回 0 台时就是 `0`（此前会回落到母鸡台数）；管理页「同步数据」提示里的退款笔数改用与退款统计页相同的口径（不再把只有 user_id、本地查不到邮箱/用户名的记录算进去）。方法名 / 参数 / 字段名都没动 |
| 2.19.19 | **行为修正，契约不变**。`vm.list` / `vm.lookup` 的 `daysLeft` 对「已过期但不到 24 小时」的实例从 `0` 改为 `-1`（此前 `Math.ceil` 得到 `-0`，而 `-0 < 0` 为假，`includeExpired=false` 拦不住它，`expiringDays` 会把刚过期的实例列成「即将到期」）；被墙检测结果里的 IPv6 地址改为原始写法（此前被当成带端口切掉最后一组，或展开成十进制 8 组）。方法名 / 参数 / 字段名都没动 |
| 2.19.18 | **行为修正，契约不变**。`vm.list` 带 `includeArchived=true` 时不再把在线实例重复列两遍（归档里同时存着在线 VM，以前每台会返回两条、total 翻倍）；其余方法名 / 参数 / 字段名都没动 |
| 2.19.17 | **无变更**。本版是主动审计：收入明细增量水位不再被同批更新的交易抬高（同轮成交多笔时较早那笔会被漏计）、今日交易与累计提现的翻页改用 pageIsLast（平台把 page_size 压到 60，旧判据只翻第一页）、黑名单越界页不再显示「黑名单为空」；RPC 方法名、参数、返回字段都没动 |
| 2.19.16 | **无变更**。本版是主动审计：容器类接口的上游 5xx 不再透传成 502（改为 422，避免面板网关替换响应体）、toast 溢出清理连节点一起删、错误文案统一走 errMsg；RPC 方法名、参数、返回字段都没动 |
| 2.19.15 | **无变更**。本版只改 VM 归档的刷新链路：交易推送不再用归档旧快照覆盖到期日、局部同步不再清空套餐名、退款标记 1 小时自愈、`lastSeenAt` 只由扫描推进；RPC 方法名、参数、返回字段都没动 |
| 2.19.14 | **无变更**。本版只改前端样式：VM 归档表加 `min-width` 兜底并给「机器」列显式宽度，修手机窄屏下 `table-layout:fixed` 把该列压成 0 宽度导致整列消失；RPC 方法名、参数、返回字段都没动 |
| 2.19.13 | **无变更**。本版只改被墙检测的判定口径：停更阈值下限 10 分钟、窗口内成功过即判通，RPC 方法名、参数、返回字段都没动 |
| 2.19.12 | **无变更**。本版只改被墙检测内部：新增数据源诊断（ok / empty / fail 三态）、tags 解析加固、任务勾选诊断与 `public:getPingRecords` 回退，RPC 方法名、参数、返回字段都没动 |
| 2.19.11 | **无变更**。本版改的是页面交互（流水筛选按钮、收入明细默认天数）与机器类型标归一化，不涉及对外 RPC 的方法名、参数或返回字段 |
| 2.19.10 | **无变更**。新增 HTTP 路由 `POST /narwhal-finance/alert-dismiss`（告警手动关闭） |

接口契约（方法名、action 名、字段名）保持向后兼容：新增字段只增不改，不会删已有键。若将来必须破坏性变更，会在版本号第二位递增并在此表登记。

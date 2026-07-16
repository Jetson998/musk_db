# NewAPI 现成能力盘点 & 迪拜中转站开发清单

> 数据来源：QuantumNous/new-api 主分支源码逐文件核对（Dockerfile / docker-compose.yml / web/default/src/features/* / model/channel.go / i18n/languages.ts / features/system-settings/types.ts）。结论按「开箱即用 ✅ / 需配置 ⚙️ / 需开发 🛠️」标注。

---

## 一、部署与运行时（官方镜像 `calciumion/new-api:latest`）

| 能力 | 状态 | 说明 |
|---|---|---|
| Docker 单镜像 | ✅ | 镜像 `calciumion/new-api:latest`，端口 3000，`WORKDIR /data` |
| SQLite 默认存储 | ✅ | 默认开箱即用，`-v ./data:/data` 持久化 |
| MySQL / PostgreSQL | ✅ | `SQL_DSN` 环境变量切换（PG 为 `postgresql://...`，MySQL 为 `root:pwd@tcp(host)/db`） |
| Redis | ✅ | `REDIS_CONN_STRING`，多节点缓存/限流必需 |
| ClickHouse 日志库 | ✅ | `LOG_SQL_DSN` 可独立日志库 + TTL |
| docker-compose | ✅ | 仓库自带 `docker-compose.yml`（含 postgres+redis，注释里给 MySQL/ClickHouse 方案） |
| 多节点部署 | ✅ | `SESSION_SECRET` + Redis 共享会话 |
| 健康检查 | ✅ | compose 内置 `GET /api/status` 探活 |
| 日志目录 | ✅ | `command: --log-dir /app/logs`，`-v ./logs:/app/logs` |
| 错误日志开关 | ✅ | `ERROR_LOG_ENABLED` |
| 批量更新 | ✅ | `BATCH_UPDATE_ENABLED` 降低 DB 压力 |
| 流式超时调优 | ✅ | `STREAMING_TIMEOUT`、`RELAY_IDLE_CONN_TIMEOUT` |
| 前端双主题 | ✅ | `web/default`（新 UI，TanStack Router + base-ui）与 `web/classic`（旧版），二进制内置两套，后台 `theme.frontend` 切换 |

> 结论：部署层 0 开发量，直接用镜像 + compose。

---

## 二、前端定制能力（你说的「优先首页」，重点看这里）

### 2.1 站点品牌字段（后台「系统设置 → 站点 → 系统信息」直接填，不改码）

| 字段 | 作用 |
|---|---|
| `SystemName` | 站点名 |
| `Logo` | Logo 图片 URL |
| `Footer` | 页脚（支持 HTML） |
| `About` | 关于页内容 |
| `HomePageContent` | **首页内容**：留空=默认落地页；填 Markdown=渲染 MD；填 HTML=渲染 HTML；填 URL=iframe 套一个外部落地页 |
| `ServerAddress` | 站点地址（回调/邮件用） |
| `theme.frontend` | 切 default / classic 两套前端 |
| `legal.user_agreement` / `legal.privacy_policy` | 用户协议 / 隐私政策文本 |

> 源码佐证：`web/default/src/features/system-settings/site/section-registry.tsx` 把这些字段注册在 `SystemInfoSection`；首页逻辑在 `features/home/index.tsx`——**留空时渲染内置 `Hero/Stats/Features/HowItWorks/CTA` 五段式落地页，填了内容就按 MD/HTML/iframe 渲染**。

### 2.2 内容/运营字段（后台「系统设置 → 内容」）

公告 `Notice`、API 信息、FAQ、聊天对话框 `chat dialog`、画图设置、控制台看板、Uptime Kuma 监控挂载——全部后台可视化配置 ✅。

### 2.3 导航/侧栏

`HeaderNavModules`（顶栏模块）+ `SidebarModulesAdmin`（侧栏模块）后台拖拽配置，可显隐菜单项 ✅。

### 2.4 自定义首页落地页结论

- 想要迪拜本地化首页：**把 HomePageContent 填一段阿拉伯语+英语的 HTML 落地页即可，0 源码改动**（最快）。
- 想要完全独立的品牌站（独立 React/SvelteKit 落地页）：HomePageContent 填那个站的 URL，用 iframe 套进来；后端 API 仍走 NewAPI。
- 想深度改内置五段式（Hero/Stats…）的样式/文案：才需要动 `features/home/components/sections/*` 源码并自构建前端覆盖（见第四节）。

---

## 三、计费 / 货币 / 充值（关键：迪拜用 AED + 充值码）

### 3.1 货币与额度显示（AED 完全可配，0 改码）

| 字段 | 作用 |
|---|---|
| `QuotaPerUnit` | 1 货币单位 = 多少额度（默认 500000） |
| `USDExchangeRate` | USD 汇率 |
| `DisplayInCurrencyEnabled` | 开关：显示为货币金额而非裸额度 |
| `general_setting.custom_currency_symbol` | **自定义货币符号 → 填 "AED" 或 "د.إ"** |
| `general_setting.custom_currency_exchange_rate` | 自定义汇率 |
| `general_setting.quota_display_type` | 额度显示类型 |

> 源码佐证：`features/system-settings/types.ts` 的 `BillingSettings`。**AED 显示是配置项，不是开发项。**

### 3.2 计费模型

`ModelRatio`（输入倍率）、`CompletionRatio`（输出倍率）、`CacheRatio`/`CreateCacheRatio`（缓存命中计费）、`ImageRatio`、`AudioRatio`、`ModelPrice`（按次固定价）、`GroupRatio`/`TopupGroupRatio`（分组倍率）、`billing_setting.billing_mode` / `billing_expr`（自定义计费表达式）——全部后台配 ✅。

### 3.3 支付网关（已内置多个）

| 网关 | 状态 | 备注 |
|---|---|---|
| EPay 易支付 | ✅ | `EpayId/EpayKey/Price/MinTopUp/PayMethods/CustomCallbackAddress` |
| Stripe | ✅ | `StripeApiSecret/StripePriceId/StripeUnitPrice/StripeMinTopUp/StripePromotionCodesEnabled` |
| Creem | ✅ | `CreemApiKey/CreemProducts` |
| Waffo | ✅ | **MENA 区域收单**：`WaffoApiKey/WaffoMerchantId/WaffoCurrency/WaffoUnitPrice/WaffoMinTopUp/WaffoPayMethods` + Pancake 分支 |
| 合规确认门 | ✅ | `payment_setting.compliance_confirmed/compliance_terms_version/...` ——开启支付前强制 KYC/合规确认 |
| 充值金额档位/折扣 | ✅ | `payment_setting.amount_options/amount_discount` |

> 即便现在走充值码，将来接 Waffo（MENA 收单）也是现成的，几乎无开发量。

### 3.4 兑换码 / 充值码（你要的，完全内置）

`features/redemption-codes`：状态 Unused/Disabled/Used，批量生成、设面额、启用/禁用 ✅。**0 开发量。**

### 3.5 钱包/签到

`features/wallet`（钱包余额）、`features/console/topup`（充值页）、`checkin_setting.enabled/min_quota/max_quota`（每日签到送额度）——全部内置 ✅。

---

## 四、模型 / 渠道 / 出海反代（两地分层的关键）

### 4.1 渠道(Channel)现成能力（`model/channel.go` + `features/channels`）

| 能力 | 状态 |
|---|---|
| **每渠道自定义上游 `base_url`** | ✅ 渠道表 `BaseURL` 字段 → 可指向 HK/SG 反代地址，无需改码 |
| 出站代理 `setting.proxy` | ✅ |
| 模型映射 `model_mapping` | ✅ |
| 参数覆写 `param_override` / 请求头覆写 `header_override` | ✅ |
| 状态码映射 `status_code_mapping` | ✅ |
| 渠道加权随机 `weight` / 优先级 `priority` | ✅ |
| 故障自动禁用 `auto_ban` + 失败重试 | ✅ |
| 多 Key 轮询/随机 `multi_key_mode` | ✅ |
| 渠道余额查询 | ✅（配合 new-api-key-tool） |
| 渠道分组 `group` + `tag` | ✅ |

### 4.2 上游协议（开箱支持）

OpenAI（含 Responses / Realtime API）、Azure OpenAI、Claude Messages、Google Gemini、Rerank(Cohere/Jina)、Midjourney-Proxy、Suno、DeepSeek、Qwen 等。格式互转：OpenAI⇄Claude、OpenAI→Gemini、Gemini→OpenAI、thinking-to-content ✅。reasoning effort 后缀（`-high/-medium/-low/-thinking`）✅。

### 4.3 两地分层结论（无开发量，纯配置）

- **方案 A（推荐）**：NewAPI 核心跑迪拜节点，每个渠道 `base_url` 填 HK/SG 反代地址。用户面在迪拜、出海经 HK/SG。
- **方案 B**：NewAPI 跑 HK/SG，迪拜只放 CDN/边缘反代或一个静态落地页（HomePageContent 填 URL）。
- 两地都靠「渠道 base_url + 反代」实现，**NewAPI 不改码**。HK/SG 上用 Nginx 反代 `api.openai.com / api.anthropic.com / googleapis.com`（注意 Host 头、SNI、SSE 关 `proxy_buffering`）。

---

## 五、用户 / 鉴权 / 统计（全部内置）

| 能力 | 状态 |
|---|---|
| 用户管理、分组、令牌分组、令牌模型限制 | ✅ `features/users` + `features/keys` |
| 用量日志 `features/usage-logs`、数据看板 `features/dashboard`、性能指标 `features/performance-metrics` | ✅ |
| 排行榜 `features/rankings`、定价页 `features/pricing`、Playground `features/playground` | ✅ |
| 登录方式：账密、Telegram、Discord、LinuxDO、OIDC | ✅ |
| 邮件 SMTP（注册/通知） | ✅ `SMTPServer/Port/Account/From/Token/SSL/StartTLS` |
| 订阅 `features/subscriptions` | ✅ |
| 关于/法律页 `features/about` `features/legal` | ✅ |

---

## 六、多语言 i18n（这里是真正的开发缺口）

| 现状 | 结论 |
|---|---|
| 内置语言 | zh-CN / zh-TW / en / fr / ru / ja / vi |
| **阿拉伯语** | 🛠️ **未内置**（`i18n/locales/` 无 `ar.json`，`languages.ts` 的 `INTERFACE_LANGUAGE_OPTIONS` 无 `ar`） |
| RTL（`dir=rtl`） | 🛠️ **未内置**（base-ui 默认主题为 LTR，无 RTL 切换） |

源码佐证：`web/default/src/i18n/languages.ts`、`i18n/locales/` 目录列表。

---

# 🛠️ 要开发/要做的事（去重后，只剩这几项）

绝大多数能力 NewAPI 已内置，真正要动手的只有「迪拜本地化」相关的少量前端 + 配置工作：

## P0 — 必须做（前端）

1. **阿拉伯语 locale + RTL**
   - 新增 `web/default/src/i18n/locales/ar.json`（从 `en.json` 翻译）
   - `languages.ts` 加 `{ code: 'ar', label: 'العربية' }`，`normalizeInterfaceLanguage` 加 `ar` 分支
   - 全局 RTL：`<html dir="rtl">` 跟随语言切换（改 `__root.tsx` 或顶层 layout），并把 CSS 改逻辑属性（`margin-inline` 等）或加 `[dir=rtl]` 覆盖
   - 这是唯一需要自构建前端并覆盖镜像内置 `dist` 的改动

2. **迪拜落地页内容（阿语+英语）**
   - 最轻：把阿语+英语 HTML 写进后台 `HomePageContent` 字段 → **0 改码**
   - 进阶：若要深度定制 Hero/Features/CTA 文案与配色，改 `features/home/components/sections/*` 后自构建覆盖

3. **货币显示 AED**
   - 后台设 `DisplayInCurrencyEnabled=true`、`custom_currency_symbol="AED"`、`custom_currency_exchange_rate` 按汇率 → **0 改码**，仅验收格式

## P1 — 配置/运营（无代码，但要做）

4. **两地分层网络**：起 HK/SG Nginx 反代上游模型；NewAPI 渠道 `base_url` 指向反代域名
5. **充值码体系**：后台 `redemption-codes` 批量生成面额码，线下/卡密分发；定价倍率 `ModelRatio/GroupRatio` 配好
6. **合规文案**：`legal.user_agreement` / `privacy_policy` 填阿语+英语版本；`ServerAddress`/Logo/Footer 配齐
7. **登录方式**：要不要开 Telegram（迪拜/海湾用户常用）——后台开关即可，0 改码

## P2 — 可选增强

8. 接 Waffo（MENA 收单）替代纯充值码——字段已内置，按需开启 + 合规确认门
9. 域名、TLS、CDN、`SESSION_COOKIE_SECURE` + `SESSION_COOKIE_TRUSTED_URL` 安全加固

## 明确「不要重复开发」的黑名单

❌ 不要自己写：兑换码、用户/令牌/分组、计费倍率、渠道管理与多 Key 轮询、用量日志、数据看板、定价页、排行榜、Playground、聊天页(`/chat2link`)、签到、订阅、Stripe/EPay/Waffo 支付、各上游协议适配、格式互转、Realtime/Responses 支持——**这些 NewAPI 全有**。

---

## 附：前端自构建覆盖镜像内置 dist 的可行方式

镜像 `WORKDIR /data`、前端 dist 在构建期被 Go embed 进二进制，**不能简单挂载覆盖**。要改前端必须：
- clone 仓库 → 改 `web/default` → 本地 `docker build` 出自己的镜像（Dockerfile 用 bun 构建 default+classic，再 Go build）
- 或保留官方镜像不动，把品牌落地页全塞进 `HomePageContent`（HTML/URL），只做阿语 locale 包打成补丁镜像

推荐：**先用 HomePageContent(HTML) + 阿语 locale 自构建镜像** 这条最小路径上线，验证后再决定要不要深改五段式首页。

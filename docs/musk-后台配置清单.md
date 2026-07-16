# Musk · NewAPI 后台配置清单

> 配套首页 `musk-home.html` 使用。按顺序逐项在 NewAPI 后台填写，全部后台操作、0 代码。版本以 QuantumNous/new-api 主分支为准（2026-07）。
> 后台路径：以管理员登录 → 左侧 **系统设置 (System Settings)**，分 `站点 / 内容 / 通用 / 运营设置 / 计费 / 安全 / 其它` 等分区（不同版本菜单略有差异，按字段名找）。

---

## 一、站点设置（Site / 系统信息）— 首页与品牌

| 字段 | 填什么 | 说明 |
|---|---|---|
| `SystemName` | `Musk` | 顶栏品牌名、浏览器标题、邮件名 |
| `ServerAddress` | `https://db.muskapis.com` | 站点对外地址，影响回调/邮件链接，**必须带 https + 你的真实域名** |
| `Logo` | 你的 Logo 图片 URL（建议透明 PNG/SVG，放 OSS/CDN） | 顶栏 logo；留空则只显示文字 Musk |
| `Footer` | 见下方「页脚文案」 | 页脚 HTML，支持简单标签 |
| `HomePageContent` | **粘贴 `musk-home.html` 全文**（从 `<style>` 到末尾 `</div>`） | 首页落地页；留空=内置默认页，填了=我们的定制页 |
| `About` | 见下方「关于文案」 | `/about` 页内容 |
| `legal.user_agreement` | 用户协议正文 | `/legal/user-agreement` |
| `legal.privacy_policy` | 隐私政策正文 | `/legal/privacy-policy` |
| `theme.frontend` | `default` | 用新版前端（我们的首页就是为 default 主题写的） |

### 页脚文案（Footer，可直接粘贴）
```html
<div style="display:flex;flex-wrap:wrap;gap:12px;justify-content:space-between;align-items:center">
  <span>© 2026 Musk · db.muskapis.com</span>
  <span>
    <a href="/pricing">Pricing</a> ·
    <a href="/console">Console</a> ·
    <a href="/about">About</a> ·
    <a href="/legal/user-agreement">Terms</a> ·
    <a href="/legal/privacy-policy">Privacy</a>
  </span>
</div>
```

### 关于文案（About，Markdown）
```md
# About Musk

Musk is a unified AI API gateway deployed in Dubai, offering OpenAI-compatible access
to the world's leading models — GPT, Claude, Gemini and more — through a single endpoint.

- **One API, every model** — switch models by changing one string
- **Usage-based billing** — pay per token in USD, no subscriptions
- **Redemption top-up** — add balance instantly with codes, no payment gateway required
- **Transparent logs** — every request's usage and cost visible in your console

Endpoint: `https://db.muskapis.com/v1`

Contact your operator for redemption codes and enterprise access.
```

---

## 二、运营设置（Operations）— 注册/登录

| 字段 | 填什么 | 说明 |
|---|---|---|
| 允许新用户注册 | 开（按需） | 关闭则只能邀请注册 |
| 邮箱验证 | 建议开 | 注册需邮箱验证，降低滥用 |
| Telegram 登录 | 选配 | 海湾用户常用 TG，要开需先 @BotFather 建 bot 并 `/setdomain` |
| Discord / LinuxDO / OIDC | 选配 | 视客群决定，默认可不开 |
| Turnstile / 验证码 | 建议开 | 防刷注册（Cloudflare Turnstile 免费） |
| `SMTPServer/Port/Account/From/Token` | 你的 SMTP | 开了邮箱验证/通知就必须配 |

---

## 三、计费设置（Billing / Pricing）— 美元计费核心

| 字段 | 填什么 | 说明 |
|---|---|---|
| `DisplayInCurrencyEnabled` | **开** | 余额/消费显示为货币金额而非裸额度 |
| `general_setting.custom_currency_symbol` | `USD` 或 `$` | 货币符号，我们用美元 |
| `general_setting.custom_currency_exchange_rate` | `1`（或按你的结算口径） | 自定义汇率 |
| `QuotaPerUnit` | `500000`（默认） | 1 美元 = 500000 额度，NewAPI 内部换算用 |
| `USDExchangeRate` | `1` | 美元汇率 |
| `general_setting.quota_display_type` | 按 UI 提示选 | 额度显示类型 |

### 模型定价（重点，逐个模型配）
进 **系统设置 → 计费/定价**，对你开通的每个模型设倍率（决定最终单价）：

- `ModelRatio`：输入倍率（相对基准 $0.002/1K tokens 的倍数）
- `CompletionRatio`：输出倍率（通常高于输入）
- `ModelPrice`：按次计费的模型填固定价（如 `gpt-image-2` 图片生成按张计费）
- `GroupRatio`：用户分组倍率（普通用户 1.0，可做大客户折扣组 0.9 等）

> ⚠️ 定价是你这单的利润来源，建议按各模型上游成本 × (1 + 目标毛利) 倒推倍率。首页不写死单价，全部交给 `/pricing` 页（由这些倍率驱动）。

### 充值码面额与折扣
- `MinTopUp`：最小充值额（如 `1` 美元）
- `payment_setting.amount_options`：充值档位（如 5/10/50/100 USD）
- `payment_setting.amount_discount`：充值赠送（如充 100 送 10）
- `TopUpLink`：外部充值链接（走充值码可留空，或指向你卖码的页面）

---

## 四、兑换码（Redemption Codes）— 你的收款方式

路径：**左侧菜单 → 兑换码（Redemption）**

操作：
1. **批量生成**：选面额（如 $5/$10/$50）、数量、有效期 → 生成
2. **分发**：导出码列表 → 线下/卡密渠道卖给用户
3. 用户在前端 **控制台 → 充值（`/console/topup`）** 粘贴码 → 余额到账

> 这是 Musk 的主要收款路径。0 在线支付、0 KYC 摩擦、0 拒付风险。

---

## 五、渠道配置（Channels）— 接上游官方 Key

路径：**左侧菜单 → 渠道（Channels）→ 新建渠道**

对每个上游厂商建渠道：

| 上游 | 渠道类型(Type) | 关键字段 |
|---|---|---|
| Anthropic | Claude | `Key` 填官方 sk-ant-...；模型列表填 claude-fable-5/opus-4-8/... |
| OpenAI | OpenAI | `Key` 填 sk-...；模型列表填 gpt-5.6-sol/terra/luna/5.5/5.4/5.4-mini/gpt-image-2 |
| Google | Gemini | `Key` 填 AIza...；模型列表填 gemini-3.5-flash/3.1-pro-preview/... |
| DeepSeek | DeepSeek | 对应 deepseek-v4-pro |
| 智谱 | Zhipu | glm-5.2/glm-5.1 |
| 月之暗面 | Moonshot | kimi-k2.6 |
| 阿里 | 通义/DashScope | qwen3.7-max |

通用要点：
- `base_url`：**留空用官方默认**（你说直接接官 key，不用反代）
- `models`：与后台计费里设的模型名**完全一致**（否则计费/路由对不上）
- `group`：默认 `default`；大客户可建单独分组走不同渠道/倍率
- `weight`/`priority`：同模型多渠道时设权重做负载/容灾
- `auto_ban`：开（渠道报错自动禁用，避免持续失败计费）
- `test_model`：填一个便宜模型（如 claude-haiku-4-5）用于「测试渠道」按钮

> 建议每个厂商至少配 2 个 key（不同账号）做容灾，靠 `weight` + `auto_ban` 自动切换。

---

## 六、安全与生产加固

| 字段 | 填什么 | 说明 |
|---|---|---|
| `SESSION_SECRET` | 随机长字符串 | 多节点/防会话伪造，**必改默认** |
| `SESSION_COOKIE_SECURE` | `true` | HTTPS 下 Secure cookie |
| `SESSION_COOKIE_TRUSTED_URL` | `https://db.muskapis.com` | 可信入口 |
| 令牌默认额度限制 | 按需 | 防一个 key 跑爆余额 |
| 用户级速率限制 | 按需 | 防刷 |
| 域名 TLS | Cloudflare/反代终止 HTTPS | 必须 HTTPS |
| `NODE_NAME` | `musk-dubai-1` | 多节点时审计日志区分 |

---

## 七、上线自检（按这个顺序）

1. **部署镜像**：`docker-compose up -d`（镜像 `calciumion/new-api:latest`，端口 3000）
2. **首次登录** root（初始管理员）→ 立即改密码
3. 填完上面 **一/三** 两节（站点 + 计费）→ 保存
4. 贴 `HomePageContent`（`musk-home.html`）→ 访问 `https://db.muskapis.com/` 看首页
5. 配 **五** 渠道 → 每个渠道点「测试」按钮确认上游 key 通
6. 配 **四** 生成首批充值码 → 自测一张能否充值到账
7. 用一张码建测试用户 → 建令牌 → 用首页 `curl` 示例实调 `gpt-5.6-luna` 或 `claude-haiku-4-5` → 确认扣费 + `/console` 日志可见
8. 开 **二** 注册/邮箱/Turnstile → 公开

---

## 附：首页 CTA 路由对照（确保后台对应页面都在）

| 首页按钮 | 跳转 | NewAPI 原生页面 |
|---|---|---|
| Get started / Get started | `/login` | 登录注册 |
| View pricing / View full pricing | `/pricing` | 定价页（后台倍率驱动） |
| Top up | `/console/topup` | 充值页（粘充值码） |
| Open console / Go to login | `/console`、`/login` | 控制台 |

> 这些路由 NewAPI default 主题原生支持，无需额外配置。

---

## 不用碰的黑名单（别重复配/别开发）

- ❌ 阿拉伯语/RTL（你定英语，跳过）
- ❌ 在线支付网关（走充值码，Stripe/Waffo/EPay 全不接）
- ❌ 出海反代（直接官 key，`base_url` 留空）
- ❌ 自构建镜像（首页走 `HomePageContent` 字段，不动 dist）
- ❌ 用户/令牌/日志/看板/排行榜/Playground 代码（全原生）

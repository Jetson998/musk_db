# Musk 首页落地页 · 技术方案

> 品牌名 **Musk**，域名 **db.muskapis.com**，英语，美元计费，充值走 NewAPI 充值码。风格对标 dataeyes.ai/models（深色科技、模型卡片网格、单 API 卖点），卖点对标 xmax token.xmax.com（统一网关、模型路由、对比）。

---

## 一、硬约束（来自 NewAPI 源码，决定一切设计）

`HomePageContent` 字段填 HTML 时，NewAPI 走 `RichContent mode='html' htmlVariant='isolated'` 渲染（源码：`web/default/src/components/html-content.tsx`、`features/home/index.tsx`）。其行为：

1. **`<script>` 被禁** —— DOMPurify `FORBID_TAGS` 含 `script`/`base`/`embed`/`object`/`link`/`meta`。**首页不能跑任何 JS**。
2. **运行在 Shadow DOM** —— 内容被挂进 `attachShadow({mode:'open'})`，并**克隆主应用所有 `<style>/<link>`** 注入影子根，所以 NewAPI 的 Tailwind + 主题 CSS 变量在首页内**可用**。
3. **`<style>` 标签和元素 `style` 属性允许** —— 这是唯一写样式的方式。
4. **明暗主题自动同步** —— `syncDarkClass` 监听 `document.documentElement` 的 `.dark` 类，在 shadow wrapper 上镜像 `.dark`。所以用 `var(--background)` 等 token + `.dark xxx` 选择器即可自动跟随站点明暗与主题色。
5. **`<a target="_blank">` 自动加 `noopener noreferrer`**；`<iframe>` 被强 sandbox（无 `allow-same-origin`）。
6. **顶栏与页脚来自 NewAPI** —— `PublicLayout` 已渲染 `PublicHeader`（Logo/导航/主题切换/登录注册按钮）和 `Footer`（后台 `Footer` 字段）。首页 HTML 只渲染在顶栏与页脚**之间**的 `<main>` 区。所以我们的 HTML **不要再画顶栏和页脚**。

**一句话：首页 = 一段自包含 HTML（含内联 `<style>`），无 JS，用 NewAPI 的 CSS 变量自动适配明暗与品牌色，靠纯 CSS 实现轻交互，CTA 链同域路由。**

---

## 二、渲染路径选型

| 方案 | 做法 | 评价 |
|---|---|---|
| **A. HomePageContent 填 isolated HTML（推荐）** | 写一段带 `<style>` 的 HTML 贴进后台 `HomePageContent` | 0 部署、同域、自动跟随主题、CTA 直跳 `/login` `/console` `/pricing`。唯一限制：无 JS。**选这个** |
| B. HomePageContent 填 iframe URL | 独立搭一个 React/静态站，iframe 套进来 | 可跑 JS、最自由，但多一个站点要维护、跨域跳转需 postMessage、首屏多一层 |
| C. 自构建镜像改 `features/home/components/sections/*` | clone 仓库改 React 源码 | 最深度，但要维护构建流水线、升级合并冲突。非必要不上 |

**选 A**：先用 isolated HTML 上线，把 90% 的视觉效果做出来。只有当需要「动态拉后台模型列表/实时价格 tab 切换」这类必须 JS 的功能时，再升级到 B（iframe 套一个轻量 React 落地页，或直接 iframe 套 NewAPI 自带 `/pricing`）。

---

## 三、页面板块设计（对标 dataeyes + xmax）

参考 dataeyes.ai/models：深色科技、渐变 hero、模型卡片网格带能力标签、能力图标行。参考 xmax：统一网关、单 API、模型路由/对比。落到 Musk：

```
[NewAPI 顶栏 — 已有，不画]
┌─────────────────────────────────────────────┐
│ HERO                                          │
│  Musk —— Your unified gateway to the world's │
│  leading AI models. Single API. Dubai-fast.  │
│  [ Get Started → /login ] [ View Pricing → /pricing ] │
│  渐变光晕背景（var(--primary) 径向渐变）       │
├─────────────────────────────────────────────┤
│ STATS（4 个指标条）                            │
│  50+ Models · 99.9% Uptime · OpenAI-compatible │
│  · Pay-as-you-go (USD)                        │
├─────────────────────────────────────────────┤
│ MODELS（核心板块，对标 dataeyes models 网格）  │
│  分组卡片：Text / Multimodal / Image / Audio  │
│  每卡：模型名 · Provider 徽章 · 能力 tag · 一句话│
│  ⚠️ 模型清单按后台实际开通渠道填，不写空头支票    │
│  下方一行：「Full list & live pricing → /pricing」│
├─────────────────────────────────────────────┤
│ FEATURES（为什么选 Musk）                      │
│  卡片网格：OpenAI-compatible API / Multi-model│
│  routing / Usage-based billing (USD) /        │
│  Redemption-code top-up / Dubai edge          │
├─────────────────────────────────────────────┤
│ PRICING 概览（对标 xmax）                      │
│  3 档示意卡（Starter/Pro/Enterprise）+ 「完整  │
│  按量价格见定价页」→ /pricing                  │
│  ⚠️ 只放示意，真实单价由 /pricing 后台驱动     │
├─────────────────────────────────────────────┤
│ QUICK START（3 步 + 代码块）                   │
│  1. Sign up & grab your key  → /login         │
│  2. Top up with a redemption code → /console/topup │
│  3. Call the API (curl 示例，深色代码块)         │
├─────────────────────────────────────────────┤
│ FAQ（用 <details> 纯 CSS 折叠）                │
│  How do I pay? → Redemption codes (USD).      │
│  Which models? → /pricing.                    │
│  Is it OpenAI-compatible? → Yes.              │
└─────────────────────────────────────────────┘
[NewAPI 页脚 — 后台 Footer 字段，不画]
```

CTA 路由全部同域相对路径（NewAPI 同站）：`/login`、`/console`、`/console/topup`、`/pricing`、`/about`。

---

## 四、样式策略（无 JS 下做到现代感）

1. **配色全用 NewAPI 变量**，自动跟随明暗 + 站点主题色：
   - 背景层 `var(--background)` / 卡片 `var(--card)` / 边框 `var(--border)` / 弱化文字 `var(--muted-foreground)` / 强调 `var(--primary)` / 成功 `var(--success)`。
   - 深色优先：`.dark` 选择器内加强渐变与发光（`box-shadow` 用 `color-mix(in oklch, var(--primary) 30%, transparent)`）。
2. **Hero 渐变光晕**：径向渐变 `radial-gradient(...)` 叠 `var(--primary)`，`@media (prefers-reduced-motion)` 关动画。
3. **模型卡片网格**：`grid-template-columns: repeat(auto-fill, minmax(260px, 1fr))`，hover 用纯 CSS `:hover` 提升边框/阴影。
4. **轻交互用纯 CSS**：
   - FAQ 折叠 → `<details><summary>`，原生无需 JS。
   - Pricing 档位/模型分组切换 → `<input type="radio">` + `:checked ~` 兄弟选择器（CSS-only tabs）。避免依赖 JS。
   - 代码块复制按钮 → 无 JS 做不到，**不放复制按钮**，给 `curl` 文本即可（用户手选）。
5. **字体**：直接用 shadow 内继承的 `var(--font-body)`（Public Sans），代码块用 `ui-monospace, monospace`。
6. **响应式**：移动端单列，`@media (max-width: 768px)` 收敛网格与 hero 字号。
7. **滚动图标墙**（对标 dataeyes 的 provider 横滚）：纯 CSS `@keyframes` + `animation: scroll` 无限滚动，hover 暂停（`.scroll-container:hover .animate-scroll{animation-play-state:paused}`）。

---

## 五、数据来源（哪些写死、哪些联动）

| 内容 | 来源 | 说明 |
|---|---|---|
| 模型清单 | **手写静态**（按后台实际开通渠道） | 无 JS 不能 fetch `/api/models`；先按你给的渠道列表写，后续渠道增减我更新 HTML |
| 模型单价 | **不在首页写死**，只链 `/pricing` | `/pricing` 由后台 `ModelRatio/ModelPrice` 驱动，避免双源不一致 |
| 充值流程 | 链 `/console/topup` | NewAPI 原生充值码页 |
| 品牌/Logo/Footer | 后台 `SystemName/Logo/Footer` | 顶栏页脚用这些，不在首页 HTML 里重复 |
| 卖点文案 | 写死在 HTML | dataeyes/xmax 风格英文文案 |

> 模型清单这块要你给我后台实际开通的模型（GPT-4o / o系 / Claude / Gemini / DeepSeek / Qwen 等），我据实写，绝不列没接的模型。

---

## 六、交互/无-JS 的能力边界

| 想要的功能 | 能否在方案 A 实现 | 处理 |
|---|---|---|
| 响应式、明暗、主题色跟随 | ✅ | CSS 变量 + `.dark` |
| 卡片 hover、渐变、滚动墙 | ✅ | 纯 CSS |
| FAQ 折叠 | ✅ | `<details>` |
| Pricing/模型 tab 切换 | ✅（轻量） | CSS-only `:checked` tabs |
| 复制按钮、动态价格、实时余额 | ❌ | 需 JS → 留到升级方案 B |
| 同域跳转登录/控制台/定价页 | ✅ | `<a href="/login">` |

---

## 七、产出与上线流程

1. 产出文件：`./output/musk-home.html` —— 一段自包含 HTML（`<style>` + 结构，**不含 `<html>/<head>/<body>` 包裹**，因为注入到 shadow root 的 wrapper.innerHTML）。
2. 配套：`./output/musk-home-preview.html` —— 加了完整 `<!DOCTYPE html>` 外壳 + NewAPI 主题变量副本的本地预览页，方便你在浏览器直接看效果（上线时不贴这个，只贴 musk-home.html 的内容）。
3. 上线：NewAPI 后台 → 系统设置 → 站点 → 系统信息 → `HomePageContent` 字段，粘贴 `musk-home.html` 全文 → 保存 → 访问 `https://db.muskapis.com/` 即生效。
4. 同步配：`SystemName=Musk`、`Logo`（给个 URL）、`ServerAddress=https://db.muskapis.com`、`Footer`（阿/英暂只要英文）、`DisplayInCurrencyEnabled=true`（美元显示）。

---

## 八、验证方式

- **本地**：浏览器打开 `musk-home-preview.html`，切 `.dark` 类验证明暗两套；窄屏验证响应式；点 FAQ/Tab 验证纯 CSS 交互。
- **线上**：贴进后台后访问首页，确认：顶栏/页脚正常（NewAPI 提供）、主题切换按钮能联动首页明暗、三个 CTA 跳转正确、无控制台报错（script 被滤属正常）。
- **回归**：升级 NewAPI 镜像不影响（首页在 DB 的 `HomePageContent` 字段，不依赖镜像内置 dist）。

---

## 九、风险与后续升级

- **风险**：模型清单与后台不同步 → 定期按渠道更新 HTML（一次性脚本化导出也可后续做）。
- **升级触发点**：一旦要「动态模型列表 / 实时价格 tab / 复制按钮 / 余额展示」，方案 A 的无-JS 天花板就到了，切方案 B（iframe 套轻量 React 落地页，或 iframe 直接套 `/pricing`）。
- **不碰**：镜像自构建（方案 C）暂不启用，保持官方镜像 + 单字段 HTML 的最轻运维形态。

---

## 下一步

给我两样我就开工写 `musk-home.html`：
1. **后台实际开通的模型清单**（决定 MODELS 板块写哪些）；
2. **要不要 Pricing 三档示意卡**（还是只放一个「按量付费，见 /pricing」的引导条）。

风格/配色/板块结构如无异议，我按本方案直接产出 HTML。

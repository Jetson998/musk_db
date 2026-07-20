Date: 2026-07-17
Operator: Jetson998 (deployment executed by external AI/ops)
Change name: Deploy Musk landing page to Dubai server (dubai.muskapis.com)

========================================
1. 部署物（唯一文件）
========================================
仓库:   git@github.com:Jetson998/musk_db.git
分支:   main
Commit: 391ac679e7cdfcd8e1bdcf563e03c6c7bf597de2 (391ac67)
文件:   output/landing/index.html  （单文件，约 76 KB，零依赖）

说明: 纯静态 HTML，CSS/JS/logo(SVG)/favicon(data-URI) 全部内联。
无 node_modules、无构建步骤、无外部资源请求。上传这一个文件即完成部署。

获取方式（二选一）:
  git clone git@github.com:Jetson998/musk_db.git && cp musk_db/output/landing/index.html <webroot>/
  或直接把 index.html 文件拷到服务器 webroot。

========================================
2. 服务器要求（迪拜节点）
========================================
- 静态托管即可（nginx / caddy / 任意静态服务）
- 域名: dubai.muskapis.com → A 记录指向迪拜服务器
- 必须 HTTPS（页面将被 https://db.muskapis.com 以 iframe 嵌入，http 会被浏览器 mixed-content 拦截）
- 关键 header 约束（iframe 嵌入依赖，配错会白屏）:
  * 不要发 X-Frame-Options: DENY 或 SAMEORIGIN
  * 不要发含 frame-ancestors 限制的 CSP；若要收紧，用:
      Content-Security-Policy: frame-ancestors https://db.muskapis.com
- 建议: gzip on；cache 可设 max-age=300（方便后续改版生效）

nginx 参考配置:
------------------------------------------------
server {
    listen 443 ssl http2;
    server_name dubai.muskapis.com;
    ssl_certificate     /etc/letsencrypt/live/dubai.muskapis.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dubai.muskapis.com/privkey.pem;
    root /var/www/musk-landing;
    index index.html;
    gzip on; gzip_types text/html;
    add_header Content-Security-Policy "frame-ancestors https://db.muskapis.com" always;
    location / { try_files $uri /index.html; }
}
server { listen 80; server_name dubai.muskapis.com; return 301 https://$host$request_uri; }
------------------------------------------------

========================================
3. 页面会产生的全部跳转（已核对源码）
========================================
A. 出站跳转 —— 全部指向 NewAPI 网关 https://db.muskapis.com，全部 target="_top"（9 处 + 顶栏 2 处 JS 重写）:
   /login          x3  (Hero「Get started」/ Quick start 第1步 / 底部 CTA)
   /pricing        x3  (Hero「View pricing」/ Models 底部「Full list & pricing」/ Pricing 区按钮)
   /console/topup  x1  (Quick start 第2步「Top up」)
   /console        x1  (Quick start 第3步「Open console」)
   /chat2link      x1  (Hero「Try playground」)
   /login          x2  (顶栏「Sign in」「Get started」— HTML 里是 href="#"，JS 启动时重写为 NEWAPI_BASE+/login)

B. 页内锚点（不产生外跳；顶栏仅独立访问时显示，iframe 内自动隐藏）:
   #models  #routing  #pricing  #faq
   顶栏品牌 href="/" → 独立访问时回 dubai.muskapis.com 根

C. JS 行为（非跳转但涉及跨窗口）:
   - iframe 内启动时向父窗口 postMessage {type:"musk-ask-prefs"}
   - 监听父窗口(NewAPI) postMessage {themeMode} {lang} → 同步明暗/中英文
   - JS 常量 NEWAPI_BASE = "https://db.muskapis.com"（约第 422 行）。
     所有出站链接由它驱动；若网关域名变更，只改这一行。

D. 无其他外链: 无统计脚本、无 CDN、无字体外链、无图片外链。favicon 为 data-URI。

target="_top" 语义: 被 NewAPI iframe 嵌入时，点击在顶层窗口跳转
（NewAPI 的 iframe sandbox 含 allow-top-navigation-by-user-activation，已验证源码支持）。

========================================
4. 部署后联动配置（NewAPI 侧，非本服务器）
========================================
NewAPI 后台 → 系统设置 → 站点 → 系统信息 → HomePageContent 填:
    https://dubai.muskapis.com
保存后访问 https://db.muskapis.com/ 即以 iframe 呈现本页。

========================================
5. 验证清单
========================================
curl -I https://dubai.muskapis.com               # 200, text/html
curl -s https://dubai.muskapis.com | grep -c data-i18n   # 应为 135
浏览器直接访问: 页面渲染、右上角 EN/中文 与 🌙 可切换
经 db.muskapis.com 访问(配好 HomePageContent 后):
  - iframe 正常显示（不白屏 → header 配置正确）
  - NewAPI 顶栏切语言/主题 → iframe 内实时跟随
  - 点 Get started → 顶层窗口跳到 db.muskapis.com/login（不是 iframe 内跳）

========================================
6. 回滚
========================================
本页部署与 NewAPI 解耦。回滚 = NewAPI 后台清空 HomePageContent（恢复默认首页）
或替换回旧 index.html。服务器侧保留上一版文件即可:
    cp index.html index.html.bak-$(date +%Y%m%d)

Local checks: node --check 通过；i18n 135 引用 / 140 定义，0 缺失
Deployment result: (部署后填写)
Backup paths/artifacts: git commit 391ac67 即部署物快照
Open risks: dubai.muskapis.com 证书需在迪拜服务器签发；NewAPI 侧 HomePageContent 配置为手工步骤

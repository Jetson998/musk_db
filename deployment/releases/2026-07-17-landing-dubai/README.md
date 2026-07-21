Date: 2026-07-17 (updated 2026-07-20: domain unified to Dubai.muskapis.com, Playground CTA removed)
Operator: Jetson998 (deployment executed by external AI/ops)
Change name: Deploy Musk landing page on Dubai server, same domain as NewAPI (Dubai.muskapis.com)

========================================
0. 架构（重要，先读）
========================================
同一台迪拜服务器、同一域名 Dubai.muskapis.com 承载两样东西:
  a) NewAPI 网关 (Docker, 127.0.0.1:3000) → 占据根路径 / (/login /pricing /console 等全是它的路由)
  b) 本静态落地页 → 挂在子路径 /landing/ (静态文件，不能占根路径，会跟 NewAPI 冲突)
NewAPI 后台 HomePageContent 填 https://Dubai.muskapis.com/landing/
访问根域名时 NewAPI 首页以 iframe 呈现落地页。同域部署，无跨域问题。

========================================
1. 部署物（唯一文件）
========================================
仓库:   git@github.com:Jetson998/musk_db.git
分支:   main
文件:   output/landing/index.html  (单文件，约 75 KB，零依赖)

说明: 纯静态 HTML，CSS/JS/logo(SVG)/favicon(data-URI) 全部内联。
无 node_modules、无构建步骤、无外部资源请求。上传这一个文件即完成部署。

获取方式:
  git clone git@github.com:Jetson998/musk_db.git
  cp musk_db/output/landing/index.html /var/www/musk-landing/index.html

========================================
2. 服务器要求（迪拜节点）
========================================
- 域名: Dubai.muskapis.com → A 记录指向迪拜服务器 (主机名大小写不敏感)
- 必须 HTTPS (Let's Encrypt 即可)
- 关键 header 约束 (iframe 嵌入依赖，配错会白屏):
  * /landing/ 路径不要发 X-Frame-Options: DENY
  * 同域嵌入，X-Frame-Options: SAMEORIGIN 可用；或 CSP:
      Content-Security-Policy: frame-ancestors 'self'
- 建议: gzip on；cache 可设 max-age=300

nginx 参考配置 (NewAPI + 落地页同域):
------------------------------------------------
server {
    listen 443 ssl http2;
    server_name Dubai.muskapis.com;
    ssl_certificate     /etc/letsencrypt/live/dubai.muskapis.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dubai.muskapis.com/privkey.pem;
    gzip on; gzip_types text/html application/json;

    # 静态落地页 (子路径)
    location /landing/ {
        alias /var/www/musk-landing/;
        index index.html;
        add_header Content-Security-Policy "frame-ancestors 'self'" always;
        try_files $uri $uri/ /landing/index.html;
    }

    # NewAPI 网关 (根路径，含 SSE 流式)
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto https;
        proxy_http_version 1.1;
        proxy_buffering off;          # SSE 流式必需
        proxy_read_timeout 300s;
    }
}
server { listen 80; server_name Dubai.muskapis.com; return 301 https://$host$request_uri; }
------------------------------------------------

========================================
3. 页面会产生的全部跳转（已核对源码，Playground 已移除）
========================================
A. 出站跳转 —— 全部指向 https://Dubai.muskapis.com (NewAPI 路由)，全部 target="_top" (8 处 + 顶栏 2 处 JS 重写):
   /login          x3  (Hero Get started / Quick start 第1步 / 底部 CTA)
   /pricing        x3  (Hero View pricing / Models 底部 Full list and pricing / Pricing 区按钮)
   /console/topup  x1  (Quick start 第2步 Top up)
   /console        x1  (Quick start 第3步 Open console)
   /login          x2  (顶栏 Sign in / Get started — HTML 里是 href="#"，JS 启动时重写为 NEWAPI_BASE+/login)
   注: /chat2link Playground 按钮已按需求删除。

B. 页内锚点 (不产生外跳；顶栏仅独立访问时显示，iframe 内自动隐藏):
   #models  #routing  #pricing  #faq
   顶栏品牌 href="/" → 回 Dubai.muskapis.com 根 (即 NewAPI 首页)

C. JS 行为 (非跳转但涉及跨窗口):
   - iframe 内启动时向父窗口 postMessage {type:"musk-ask-prefs"}
   - 监听父窗口(NewAPI) postMessage {themeMode} {lang} → 同步明暗/中英文
   - JS 常量 NEWAPI_BASE = "https://Dubai.muskapis.com" (约第 421 行)。
     所有出站链接由它驱动；若网关域名变更，只改这一行。

D. 无其他外链: 无统计脚本、无 CDN、无字体外链、无图片外链。favicon 为 data-URI。

========================================
4. 部署后联动配置（NewAPI 后台）
========================================
系统设置 → 站点 → 系统信息 → HomePageContent 填:
    https://Dubai.muskapis.com/landing/
保存后访问 https://Dubai.muskapis.com/ 即以 iframe 呈现本页。

========================================
5. 验证清单
========================================
curl -I https://Dubai.muskapis.com/landing/            # 200, text/html
curl -s https://Dubai.muskapis.com/landing/ | grep -o data-i18n | wc -l   # 应为 245 (i18n 埋点出现次数; 对应 134 个不同 key)
curl -s https://Dubai.muskapis.com/api/status | grep -o success   # NewAPI 存活
浏览器直接访问 /landing/: 页面渲染、右上角 EN/中文 与月亮图标可切换
访问根域名 (配好 HomePageContent 后):
  - iframe 正常显示 (不白屏 → header 配置正确)
  - NewAPI 顶栏切语言/主题 → iframe 内实时跟随
  - 点 Get started → 顶层窗口跳到 /login (不是 iframe 内跳)

========================================
6. 回滚
========================================
本页部署与 NewAPI 解耦。回滚 = NewAPI 后台清空 HomePageContent (恢复默认首页)
或替换回旧 index.html。服务器侧保留上一版文件即可:
    cp index.html index.html.bak-$(date +%Y%m%d)

Local checks: node --check 通过；i18n 134 引用 / 139 定义，0 缺失
Deployment result: (部署后填写)
Backup paths/artifacts: git main 分支即部署物快照
Open risks: 证书需在迪拜服务器签发；NewAPI 侧 HomePageContent 配置为手工步骤

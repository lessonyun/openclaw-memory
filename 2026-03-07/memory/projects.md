# 项目信息

## Discord 频道映射
- #多多 (主频道): 1468468699491995761
- #牛牛: 1468468738583040021
- #一起闲聊: 1468324585983316277
- #加仓趋势提醒: 1468471411071783124
- #微信插件开发: 1471046557901131979
- #企业微信插件开发: 1473865132705583124
- #中控面板开发: 1474325737451421706
- #哥飞出海学习: 1475079862204235880
- #视频内容分析: 1475084270656557127

## 任务队列系统
- 任务文件: workspace/tasks.md
- cron 每5分钟扫描 pending 任务
- 任务完成后通知到指定 Discord 频道
- 所有 agent/session 共享同一个 tasks.md

## 饿了么联盟Cookie管理 (ele-cookie-manager)
- 线上: http://api.52xnz.cn | 服务器: /www/ele-cookie-manager | 本地: workspace/ele-cookie-manager
- 技术栈: Express + puppeteer-core + sql.js + systemd (ele-cookie)
- Cookie上传: POST https://api.wandanlianmeng.com/api/updateCookies?key=d67cadac2901b9d871422d13d937c287 `{id, cookie}`
- Check URL: `union.ele.me/index#/alliance-child-effect-report/promotion` (302=expired)
- 浏览器缓存: data/profiles/account_{id}/ | 截图: data/screenshots/account_{id}.png
- 部署: sftp上传tar+ssh解压，Nginx反代3000端口
- 登录: iframe(ipassport.ele.me)内填表单→勾协议→登录→可能要SMS
- ext_id: 万单联盟的外部ID

## 推客密码 (tuikemima.com)
- 账号: 17638421352 / adsads123 | 渠道ID: 94525 | 用户名: fk_94525
- 公司: 郑州琪岸网络科技有限公司 | 银行: 浙江网商银行 8888888474498704
- **对公提现流程**: 登录→财务管理→对公佣金提现→对公提现按钮→全部→选公司→上传发票→确定
- 技术: Ant Design，用 CDP `DOM.setFileInputFiles` 注入文件上传

## 微信域名监控 (wx.52xnz.cn)
- 部署: /www/wwwroot/wx-domain-monitor，端口3002，systemd: wx-monitor
- 功能: 公众号接口检测链接封禁 + 企微Webhook通知
- 登录: lessonyun / adsads123 | Cookie会过期需重新扫码

## 饿了么口令数据大屏 (kouling.52xnz.cn)
- 部署: /www/wwwroot/ele-dashboard，端口3003，systemd: ele-dashboard
- 数据源: 饿了么联盟API 17638421352 | 登录: lessonyun / adsads123

## 抖音视频分析系统
- 已迁移到服务器: 120.53.92.153:18810（Docker容器，自动重启）
- 项目: /www/douyin-api | API: github.com/Evil0ctal/Douyin_TikTok_Download_API
- 待办: 配置 cookies，开放安全组端口

## AI 监控大屏 (ai.52xnz.cn)
- 静态: /www/ai-dashboard/ | API: port 3001, systemd: ai-dashboard-api
- **v4 重写完成 (2026-02-24)**: 浅色现代风，折叠/搜索/导航功能
- **新架构**: 前端(北京 PM2 command-center:3001) + API(Mac mini PM2 api:3901)
- **API 公网**: https://zk.yhnas.52xnz.cn:8848 → Lucky 反代 → 192.168.2.143:3901
- 登录: lessonyun / adsads123
- 数据库: 腾讯云 MySQL, ai_dashboard / C2ePtC7Zz5skLBbT
- 表: agent_status, agent_tasks, agent_activities, agent_skills
- 多多脚本: scripts/report_status.py | 本地: workspace/ai-dashboard/

## 鲜花投放监控 (xianhua.52xnz.cn) — 已结束
- 部署: /www/wwwroot/xianhua-dashboard，端口3004
- 数据源: 美团联盟x2 + 巨量引擎 | cron 已停

## Open-AutoGLM (手机AI操控)
- 路径: workspace/Open-AutoGLM
- API: https://open.bigmodel.cn/api/paas/v4, model: autoglm-phone
- ADBKeyboard 已启用 | Launch action 只能开预设 app

## Twitter 每日监控
- **脚本**: `scripts/twitter-daily-summary.sh`
- **数据目录**: `data/twitter-summaries/YYYY-MM-DD.json.raw.txt`
- **Cron**: `twitter-daily-summary`，每天 21:00 (Asia/Shanghai)
- **推送频道**: Discord #推特每日总结推送 (channel id: 1475929331061624832)
- **数据源**: 东京服务器 RSSHub (43.167.246.92)，via SSH + sshpass
- **监控账号**: 29个（AI大佬/中文AI圈/独立开发/加密货币 四类）
- **账号分类**:
  - AI大佬: karpathy, emollick
  - 独立开发: levelsio, steipete, gregisenberg, corbinbraun, EXM7777, vasuman, egeberkina, MengTo, rryssf, kloss_xyz, Hesamation
  - 中文AI: dotey, oran_ge, tuturetom, geekbb, vista8, op7418, seclink, xiaohu, lijigang, himself65, xiaomucrypto, Khazix0918, berryxia
  - 加密货币: okxchinese, sunyuchentron, sjhsjh0628

### 2026-02-25 优化记录
- **问题**: ①双@bug(`@@levelsio`) ②转推混入 ③低质生活流水账内容 ④摘要格式无结构约束
- **修复**:
  - 脚本改用 `account` 变量代替 `author.text`，彻底修复双@问题
  - 新增 `SKIP_RT=1` 参数，默认过滤转推
  - 新增 `MAX_PER_ACCOUNT=5` 限制每账号推文数
  - 账号列表改为 `account:category` 格式，方便分类
  - Cron 提示词全面重写：明确保留/过滤标准、每类最多3条、总量12-15条、Discord 格式模板

## 定时任务
- 价格监控: `price_hourly.py` 每小时整点
- 早间播报: `morning_report.py` 每天 10:00
- 提醒阈值: 跌幅 >= 3%

## 微信机器人插件 (wechat-plugin)
- 目录: /Users/yunhan/.openclaw/workspace/wechat-plugin/
- Web 管理面板: http://x.52xnz.cn (pm2 wechat-bot)
- **当前版本: v2.1.7**, WeChat 8.0.69 砸壳 IPA
- ObjC dylib + Node.js/Express + React
- **已完成功能**: 消息收发、图片收发、语音发送(edge-tts→silk)、防撤回、红包自动抢(两步法)、AI聊天(Sonnet 4.6直连)、远程日志、per-group @触发开关、消息合并缓冲区(4s)
- **AI架构**: 服务端直接调 Anthropic API (s2a.laolin.ai)，不通过 OpenClaw
- **搜索/GIF待完成**: 需 Mac mini 代理（国内服务器访问不了国外 API）
- 详见 memory/2026-02-19.md ~ 2026-02-22.md

## BabyNamePick (babynamepick.com)
- AI Baby Name Generator 工具站
- 免费、无需注册、20+ 文化起源、AI驱动
- **Google Analytics**: G-QHVJ2484CR (GA4, 账号ID: a386740414, 媒体资源: p527320490)
- **Vercel Token**: [REDACTED]
- SEO外链建设中（2026-03-01 启动，226条外链资源）
- 外链数据: workspace/data/backlink-sites.json
- 外链结果: workspace/data/backlink-results.json (91个成功提交)
- 19 个 SEO 分类页面 + 首页 + browse + about = 22 页
- 博客系统: Claude Code 正在开发 5 篇 SEO 文章

## 出海赚美元项目 (us.52xnz.cn)
- 状态: Phase 1 准备期（2026-02-22 启动）
- 学习报告: us.52xnz.cn | 文件: gefei-report.html
- 计划文件: memory/sea-project.md
- 学习笔记: memory/seo-study-notes.md
- 技术栈: Next.js + Tailwind + Vercel/Cloudflare
- 方法论: 哥飞精品工具页面V2.0
- 频道: Discord #哥飞出海学习
- 竞争参考: 老林已批量建站（工作流自动化）
- 我们策略: 精品路线，先做好一个站

## 闲鱼数据大屏 (xianyu-dashboard)
- **位置**: `workspace/xianyu-dashboard/`
- **技术栈**: Python/Flask + PyMySQL + Vue.js + ECharts
- **端口**: 5000（本地开发）
- **登录**: xianyu / xianyu8848
- **数据库**: 本机 MySQL 127.0.0.1:3306 / xianyu_dashboard
- **主要文件**:
  - `api_server.py` — Flask API 服务（5个接口 + 认证）
  - `db.py` — 数据库操作（upsert 函数 + `db_cursor` 上下文管理器）
  - `utils.py` — 公共工具函数（safe_int/safe_float/safe_div）
  - `sync_fans_new.py` — 从两个主体 API 同步进粉数据
  - `sync_tb.py` — 同步淘宝订单（acquire/broadcast 两模式）
  - `sync_tuike.py` — 同步推客订单（含 token 缓存 + 文件锁）
  - `sync_calc.py` — 计算衍生指标（ROI/转化率/单产）
  - `dashboard_v2.html` — 前端大屏页面
- **数据源**:
  - 主体1 另岸: http://wd2.52xnz.cn/api.php
  - 主体2 璀璨星河: http://wd.52xnz.cn/api.php
  - 淘宝: https://api.ll.vryfh.top/outlink/api/tb/ele_vip/pid/data
  - 推客: https://japi.tuikemima.com (用户 17638421352)

### 2026-02-25 优化（subagent）
1. 新增 `utils.py`：统一 safe_int/safe_float/safe_div，消除 3 个文件重复定义
2. `db.py`：新增 `db_cursor` 上下文管理器，简化连接管理
3. `api_server.py`：
   - 新增 `_get_request_date_range()` 消除 4 处参数解析重复
   - 支持 `last30days` 日期类型
   - 广播汇总 fans_total/fans_active 改为取最后一天值（累计量不应相加）
   - 全局 logger，所有异常带 exc_info 记录
4. **修复 Bug** `sync_broadcast.py`：`total_orders = tb_orders + mt_orders` 中 `mt_orders` 未定义，已改为 `tuike_orders`
5. `sync_tb.py`, `sync_tuike.py`, `sync_calc.py`：改为从 utils.py 导入工具函数

### 2026-03-04 龙虾小队安全+UI重构（PROJ-003）✅ 已验收
- **最终测试: 21/21 AC 全部通过**（千千重测 20/21 + HTTPS 已配，多多验收确认 AC4 也通过）
- **Fix1/Fix2 补测: 21/21 AC 全部通过**（数据交叉验证: total_orders=15791, total_income=10321.85，群发次数KPI卡已删）
- **已修复**: CORS 白名单、DB凭证移入 .env、README 无明文密码、Decimal→int/float 强转
- **部署 bug 修复**: python-dotenv 加入 requirements.txt、.env.example DB_USER 改为 dashboard_user、DictCursor 已加
- **新增**: 进粉趋势面积图（每日汇总+source筛选）、浅色UI、全局日期筛选、KPI汇总卡、响应式布局
- **代码**: debug=False、统一错误格式、get_conn 统一
- **HTTPS**: xianyu.52xnz.cn 已配（Let's Encrypt，证书到期 2026-06-02），HTTP→HTTPS 301 正常
- Nginx 配置: /www/server/panel/vhost/nginx/xianyu.52xnz.cn.conf

### TODO（前端）
- 详见 TODO.md

### 2026-03-04 老板新需求（子区发的）
- 日期选择要醒目，默认展示今天数据
- 近7天选总佣金不显示 — bug
- 日期区间改为单框选开始+结束日期
- 进粉数据参考 qw.pipi.pet 源码重写，用企微 cookie 直接拉数据
- 附件: qw.pipi.pet_XjNtW.zip（Discord 闲鱼数据大屏子区）
- **子区 thread ID**: 闲鱼数据大屏 1478734169269080084 / 龙虾小队作战室 1478734290870206474

---

## 东京服务器接入 OpenClaw 节点操作手册

> 调查时间：2026-02-25
> 调查者：多多（subagent）

### 服务器现状摘要

| 项目 | 值 |
|------|-----|
| 服务器名称 | 牛牛（腾讯云东京） |
| 公网 IP | 43.167.246.92 |
| 内网 IP | 10.7.0.15（hostname: VM-0-15） |
| SSH | root / cOYYKO1st.Rc |
| 操作系统 | OpenCloudOS 9.4, 2核/4GB/60GB |
| Node.js | v22.22.0（系统级 /usr/local/bin） |
| OpenClaw 版本 | v2026.2.23（已安装） |
| Gateway 状态 | ✅ 运行中，监听 127.0.0.1:18789（loopback） |
| Node service | ❌ 未安装（systemd disabled） |
| 可用内存 | 约 2.1GB |
| 磁盘剩余 | 51GB |

### 节点接入原理

`openclaw node` 是**客户端主动连接**机制：
- 东京服务器运行 `openclaw node run --host <gateway-host> --port <port>`
- 主动连接到 Mac mini 的 Gateway（`ws://` 或 `wss://`）
- Mac mini 的 Gateway 审批配对请求 → 配对成功
- 之后 Mac mini 可通过 `openclaw nodes` 命令控制东京服务器（截图、运行命令等）

**核心挑战：** Mac mini 在内网（192.168.2.143:18789），东京服务器无法直接访问。

---

### 方案 A：SSH 反向隧道（推荐 · 零配置）

**原理：** Mac mini 通过 SSH 把自己的 gateway 端口映射到东京服务器的 loopback。

**步骤 1 - Mac mini 建立反向隧道（本地执行）：**
```bash
# 持久化运行（建议用 autossh）
autossh -N -M 0 -o "ServerAliveInterval=30" -o "ServerAliveCountMax=3" \
  -R 18790:127.0.0.1:18789 root@43.167.246.92
```

或临时测试：
```bash
ssh -N -R 18790:127.0.0.1:18789 root@43.167.246.92
```

**步骤 2 - 东京服务器安装并启动 Node service（SSH 到东京执行）：**
```bash
# 安装 node 服务（连接到本机 18790 = Mac mini gateway）
openclaw node install --host 127.0.0.1 --port 18790

# 启动
openclaw node start
# 或
systemctl --user start openclaw-node.service
```

**步骤 3 - Mac mini 审批配对（本地执行）：**
```bash
openclaw nodes pending   # 查看待审批
openclaw nodes approve <node-id>  # 审批
openclaw nodes status    # 确认配对成功
```

**持久化方案 - 使用 autossh（Mac mini 添加 LaunchAgent）：**
```xml
<!-- ~/Library/LaunchAgents/com.openclaw.tokyo-tunnel.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.openclaw.tokyo-tunnel</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/autossh</string>
    <string>-N</string><string>-M</string><string>0</string>
    <string>-o</string><string>ServerAliveInterval=30</string>
    <string>-o</string><string>ServerAliveCountMax=3</string>
    <string>-R</string><string>18790:127.0.0.1:18789</string>
    <string>root@43.167.246.92</string>
  </array>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
</dict>
</plist>
```
```bash
launchctl load ~/Library/LaunchAgents/com.openclaw.tokyo-tunnel.plist
```

---

### 方案 B：Lucky 反代（更稳定，需要配置）

**原理：** Mac mini 的 Lucky 反代已有 `zk.yhnas.52xnz.cn:8848` 入口，新增一条 WebSocket 路由。

**步骤：**
1. 打开 Lucky 管理界面（本地访问）
2. 新增反代规则：
   - 协议：HTTPS+WSS
   - 外部域名：如 `oc.yhnas.52xnz.cn`（需添加 DNS A 记录）
   - 内部目标：`ws://127.0.0.1:18789`
3. 东京服务器执行：
   ```bash
   openclaw node install --host oc.yhnas.52xnz.cn --port 443 --tls
   openclaw node start
   ```
4. Mac mini 审批配对（同方案 A 步骤 3）

---

### 方案 C：Gateway bind=lan（简单但不安全）

⚠️ **不推荐**：会把 gateway 暴露在公网，需要依赖 token 认证。

东京服务器 gateway 的 bind 已设置为 `lan`，但当前以 loopback 运行。若要用此方案需开放安全组 18789 端口（当前未开放，也不建议）。

---

### 待办事项

| # | 事项 | 负责 | 状态 |
|---|------|------|------|
| 1 | 在 Mac mini 安装 `autossh`（`brew install autossh`） | 云寒确认 | ⏳ |
| 2 | 建立 SSH 免密登录（将 Mac mini 公钥加入东京 authorized_keys） | 需操作 | ⏳ |
| 3 | 启动反向隧道（方案A 步骤1） | 多多可执行 | ⏳ |
| 4 | 东京安装+启动 node service | 多多可执行 | ⏳ |
| 5 | Mac mini 审批配对 | 多多可执行 | ⏳ |
| 6 | 验证 `openclaw nodes status` 显示东京节点 online | 验收 | ⏳ |

### 已知 SSH 配置问题

- Mac mini → 东京：使用 sshpass（密码认证），见 `scripts/twitter-daily-summary.sh`
- 建议：将 Mac mini 的 SSH 公钥（~/.ssh/id_ed25519.pub 或 id_rsa.pub）追加到东京 `~/.ssh/authorized_keys`，实现免密访问（autossh 需要免密）

**检查/生成 Mac mini 公钥：**
```bash
cat ~/.ssh/id_ed25519.pub 2>/dev/null || cat ~/.ssh/id_rsa.pub 2>/dev/null || ssh-keygen -t ed25519
```

**添加到东京：**
```bash
sshpass -p 'cOYYKO1st.Rc' ssh-copy-id -o StrictHostKeyChecking=no root@43.167.246.92
```

## 海外域名
- **yunhan.io** — Cloudflare 注册+托管，用于中转站业务
- DNS: Cloudflare（自带 CDN、DDoS 防护、免费 SSL）
- 用途：AI API 中转站对外服务域名

## 🦞 龙虾小队 (Lobster Squad)
- **目标**: 多 Agent 协同 AI 公司流水线（调研→PRD→开发→测试→验收）
- **NAS**: 192.168.2.6 (飞牛NAS, Debian 12, i3-N305 8核, 48G RAM)
- **部署**: Docker Compose, 3 Agent 容器 + Hub 容器 + 共享 workspace volume
  - 🌸 花花 xiaoxia :3102 — 情报中心/调研分析
  - 💰 钱钱 xiaoxie :3103 — 产品经理
  - 🐉 小龙 xiaolong :3101 — 全栈工程师
  - 🦞 Hub lobster-hub :3100 — WebSocket 通信中枢
  - 🐂 牛牛 (东京 43.167.246.92) — 监管/测试（待接入）
- **文件**: /vol2/1001/lobster-squad/ (compose + config + hub + Dockerfile)
- **Hub**: ws://192.168.2.6:3100, Token: lobster-hub-2026
- **Bridge**: 每个容器内跑 bridge.js，连接 Hub ↔ openclaw agent CLI
- **流水线**: 花花→钱钱→小龙→(牛牛)→多多，自动 handoff 已验证
- **积分**: /workspace/scores.json, Hub API /api/score
- **计划文档**: lobster-squad/PLAN-v4-final.md
- **状态**: Phase 1 完成 — Hub+Bridge+流水线自动交接验证通过
- **下一步**: 接入牛牛、跑真实项目、Dashboard

### 龙虾小队 Discord 频道变更 (2026-03-04)
- **指挥室频道**: 龙虾小队-指挥室 (id: 1478721846517174455)
- 龙虾小队相关沟通转移到指挥室
- duoduo bridge 推送目标已改为指挥室频道
- webhook 触发 session 也改为指挥室

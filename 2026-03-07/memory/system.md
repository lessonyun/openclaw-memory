# 系统环境与经验

## 本机环境
- Mac mini (Apple Silicon, arm64), macOS Darwin 25.2.0
- 用户: yunhan (admin)
- 搜索代理: `search-proxy.js` port 18790（Brave Search API 中转）
- OpenClaw 以 launchd 服务运行 (`ai.openclaw.gateway`)
- Node: `/Users/yunhan/.nvm/versions/node/v24.13.0`
- 配置: `/Users/yunhan/.openclaw/openclaw.json`
- 模型: Claude Opus 4.6 (2026-02-09 从 4.5 升级)
- OpenClaw 版本: 2026.2.19-2（2026-02-21 升级）
- QMD 记忆后端: 已启用（`memory.backend = "qmd"`），本地嵌入
- Bun: 1.3.9（`~/.bun/bin/bun`）
- Chrome: `/Volumes/外置2t/Applications/Google Chrome.app`
- DeepReeder: `~/.agents/skills/deep-reeder/` (抓取 X/Reddit/YouTube/微博/B站等)
- 外置硬盘: 2TB，挂载在 `/Volumes/外置2t`，优先使用

## 龙虾小队 Hub API
- 地址: http://192.168.2.6:3100
- Auth: `Authorization: Bearer lobster-hub-2026`

### 🚨 龙虾小队任务流转铁律（2026-03-05 云寒指令）
**多多是 CTO，不是 PM！职责分清：**
- 多多：技术架构、Hub 开发维护、部署、浏览器验收
- 钱钱：产品经理，PRD 是她写的，任务分配是她做的
- **PRD、产品设计、任务拆解 = 钱钱的活，多多不要越权**
**多多（CTO）绝对不能直接给小龙/牛牛/花花派 task 类型消息！**
- 正确流程：多多 → 钱钱（PM）→ 钱钱分配给具体 Agent
- 结果回流：Agent → 钱钱验收 → 钱钱汇报给多多
- 多多只能给钱钱发 task/handoff
- 多多可以给任何人发 chat（聊天/问进度），但不能发 task
- 违反后果：Agent 会把 result 回给多多而不是钱钱，PM 脱离 loop，流程断裂
- **唯一例外**：紧急 bug 修复且钱钱离线时，可临时直接派，但必须事后通知钱钱补录
- 发消息: `POST /api/send` body: `{"from":"duoduo","to":"<agent>","type":"task|result|handoff","payload":{"message":"..."}}`
- 查状态: `GET /api/status`
- 查事件: `GET /api/events?limit=N`
- **注意**: `/api/message` 和 `/api/messages` 不存在，只有 `/api/send`
- **Bug记录 (2026-03-05)**: 收到 huahua 的 result 后自动 ack 时，发了一个空 result `{}` 出去。需要排查 bridge 的 ack→result 逻辑，避免空 payload。

## 已安装工具
- Homebrew 5.0.13
- ffmpeg 8.0.1, yt-dlp 2026.1.31
- Claude Code CLI: 能用，通过 s2a.laolin.ai 中转。简单任务~15s，复杂任务60-120s。**不要设 timeout，用 background:true + poll 等自然完成**。之前反复超时就是因为多余设了 timeout
- OpenAI Codex CLI (v0.104.0): 已安装可用。中转站 gmn.chuangzuoli.com，key sk-0fef...060e，模型 gpt-5.2-codex。注意：它会读 workspace 所有文件，用时限定工作目录。用了 13k tokens 做一个 hello world，比 Claude Code 贵
- TTS: edge-tts, voice: zh-CN-XiaoxiaoNeural
- **语音转录**: faster-whisper (large-v3)
  - 脚本: `scripts/transcribe.py`
  - 模型缓存: `/Volumes/外置2t/AI-Models/huggingface/` (2.9GB)
  - 软链接: `~/.cache/whisper` → `/Volumes/外置2t/AI-Models/whisper`

## 经验教训（踩坑记录）
- **截图**: 用 `/Users/yunhan/Applications/OpenClaw Screen.app/Contents/MacOS/openclaw-screen -x <output.png>`，App 包装器解决 launchd 权限问题
- **sudo**: launchd 下 elevated 不生效，用 `osascript` + `administrator privileges` 替代
- **PATH**: brew 命令前加 `export PATH="/opt/homebrew/bin:$PATH"`
- **Python 包**: 用 `brew install` 而非 `pip install`（PEP 668 限制）
- **Telegram 发送**: 插件网络不稳定时可用 curl 直接调 Bot API
- **elevatedDefault**: 保持 `"off"`
- **Discord guilds.channels**: 配了白名单则只有列出的频道生效
- **config.patch**: 无法删除已有 key，需直接编辑 JSON
- **config 修改后必须重启**: 且重启后要在 Discord 确认连接恢复
- **Gateway token 不匹配修复**: plist 会缓存旧 token，改了 config 里的 token 后 plist 不会自动同步。遇到 "gateway token mismatch" → 执行 `openclaw gateway install && openclaw gateway start`（重新生成 plist）。不要只改 config + restart。
- **Gateway lan bind + allowedOrigins**: 新版本要求 bind=lan 时必须配置 allowedOrigins，否则拒绝启动。临时修复: `dangerouslyAllowHostHeaderOriginFallback: true`。升级 OpenClaw 后如果 gateway 起不来，先查这个。
- **Notes 备忘录**: osascript 在 launchd 下可能挂起
- **SIGUSR1 热重载**: 无法加载新 npm 包代码，需完全重启 `openclaw gateway stop && start`
- **OpenClaw Webhook 端点**: Gateway 内置 `/hooks/agent` 和 `/hooks/wake`，外部程序可 POST 触发 agent 任务。需配置 `hooks.enabled: true` + `hooks.token`。这是外部系统接入 OpenClaw 的正确方式（不是 sessions REST API）
- **Cron任务必须区分是否需要LLM**: 纯执行类任务（上报、监控、脚本）用系统crontab直接跑Python/Bash脚本，零token。只有需要AI思考的任务（日报、分析、回复）才用agentTurn。**遇到X就做Y：新建cron时先问"这个任务需要AI思考吗？"不需要就用系统crontab。**
- **macOS crontab需要TCC权限**: 首次设置可能被拒绝，需要用户在系统偏好设置中授权
- **iOS 微信插件**: class-dump 可能失败用 strings 搜索；inject 工具注入 dylib；zsign 不支持中文文件名
- **iPad 协议 admin API**: 参数名是 `key` 不是 `adminKey`（看源码才发现）
- **宝塔面板 MySQL 密码**: `sqlite3 /www/server/panel/data/default.db "SELECT mysql_root FROM config LIMIT 1;"` 拿到的值可能不是真实密码（返回了 `admin` 但不对）
- **Redis 密码清除**: 必须用当前密码认证后才能 CONFIG SET requirepass ''（`redis-cli -a 'oldpass' CONFIG SET requirepass ''`）
- **Go 交叉编译**: `CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build` 在 Mac 上编译 Linux 二进制
- **OpenClaw 容器部署**: 容器内没有 systemd，gateway 启动必须用 `openclaw gateway run`（前台），不能用 `start`（依赖 systemd/launchd）。配置 `gateway.bind: "lan"` + `gateway.mode: "local"`
- **OpenClaw 配置迁移**: `defaultModel` → `agents.defaults.model.primary`; `agent.*` → `agents.defaults.*`
- **飞牛NAS 目录权限**: /opt 是 root 的，用户可写目录在 /vol2/{uid}/（如 /vol2/1001/）
- **复杂部署先写计划**: 云寒指令——先出方案文档审核再执行，减少试错成本

### ObjC/iOS 插件开发专项（血泪教训）
- **NSNumber 取值**: ❌`(long)nsNumber` 是指针转换！ ✅`[nsNumber longValue]` — NSDictionary 里存的都是对象
- **AsyncOnAddMsg 的 wrap 对象**: 图片路径属性为 nil，必须用 `CMessageMgr.GetMsg:LocalID:` 从数据库重新获取
- **WXAM 解码必须在主线程**: `dispatch_async(main_queue)` + semaphore，后台线程调用会 EXC_BAD_ACCESS
- **WXAM 解码用 objc_msgSend**: 不用 NSInvocation（ARC 下 `__unsafe_unretained` + `getReturnValue` 会野指针）
- **UIKit dlopen 后不要 dlclose**: UIKit 是常驻框架，dlclose 可能导致返回值内存被回收
- **`[data copy]` 跨线程传递**: 主线程 block 里生成的 NSData 传到其他线程前必须 copy
- **multipart/form-data 在 iOS 上不稳定**: NSURLSession 发送大 multipart body 会闪退，用 base64 JSON 更可靠
- **微信图片格式**: `.pic_thum` = JPEG（可直接用），`.pic/.pic_hd` = WXAM（需要解码）
- **WxAMAnimatedImageDecoder**: 有 2 参数和 3 参数版本，优先用 `imageWithWxAMData:scale:config:` (config 传 nil)
- **编译检查**: `grep "error:"` 返回 exit=1 表示没找到错误（grep 无匹配返回 1），不要误判为编译失败

### OpenClaw Webhook
- OpenClaw **有** 内置 webhook: `POST /hooks/agent` 和 `POST /hooks/wake`
- 配置: `hooks.enabled=true` + `hooks.token` 在 openclaw.json
- **但**: 安全沙箱会把 webhook 内容包在 `EXTERNAL_UNTRUSTED_CONTENT` 里，hook session 的 AI 会拒绝执行其中的命令（curl/exec 等）
- **不适合需要回调的场景**——webhook 是异步的（返回 runId），没有同步返回结果的机制
- 适合：通知类、deliver 到 messaging channel 的场景
- 不适合：需要 AI 执行外部操作然后回调的场景

### 给云寒发命令的格式
- **用单反引号包裹命令**（内联代码），Discord 手机端可以点击直接复制
- ✅ `command here` — 点击可复制
- ❌ ```代码块``` — 手机上不能点击复制
- ❌ components/按钮 — Discord 没有复制按钮功能
- 每条命令单独一行，说明用途

### 核心行为准则（2026-02-22 反思）
- **先验证再操作** — 不要"先试试看"，先确认环境、身份、参数是否正确
- **先看日志/文档再推测** — 不要凭经验猜，日志里有答案
- **追根因不绕过** — 遇到异常先搞清楚为什么，不要用 workaround 跳过
- **改配置前查文档** — 确认字段名有效，不要凭印象写

### 远程服务器操作铁律
- **所有业务部署都在北京 120.53.92.153**，东京 43.167.246.92 是牛牛专用
- **改 Nginx 配置必须同步宝塔面板**：不能只改配置文件，要通过宝塔面板或同时更新宝塔的站点目录/反代设置，否则宝塔显示旧信息且可能被宝塔覆盖
- **操作前必须确认 hostname**：`hostname` 命令，东京=VM-0-15，北京=VM-0-5
- **两台服务器密码不同**：东京 cOYYKO1st.Rc，北京 NIS]cOYYKO1st.Rc
- **fail2ban 风险**：多次密码错误会被 ban，连接前确保密码正确
- **路由不稳定**：Mac mini 到东京有时会路由到北京，每次连接都要验证

### NAS 备份
- 飞牛 NAS: /Volumes/云寒文件 (SMB, 192.168.2.6)
- 自动备份 cron: 每天凌晨 3:00
- rsync 在 SMB 上不稳定，用 cp -R

### Web 游戏安全测试方法论
- **遇到X就做Y**: 先抓 API 列表（grep 前端代码中的 api 调用），再逐个测试参数篡改/频率限制/越权
- claim 类 API 是重灾区——很多游戏只在前端检查条件，后端不验证
- 注册+邀请系统组合漏洞是最常见的刷钱方式

### 远程服务器文件编辑
- 遇到X: 用 heredoc/ssh 写含模板字符串(`${var}`)的JS文件 → 做Y: 本地写好再 scp
- zsh 会展开 `${var}`、反引号等，导致文件内容被破坏

### 开发流程原则
- **先研究参考代码再动手** — 有参考实现时完整照抄，不要猜
- **日志一次加够** — 每个异步操作的触发、回调、成功/失败都加 remoteLog
- **写完代码逐行审查** — 重点检查类型转换、nil 安全、线程安全、变量作用域
- **不要拿用户当调试器** — 每次发版前自己过一遍所有代码路径

### 国内服务器（腾讯云）网络限制
- **无法直接访问**: Brave Search API、wttr.in、Tenor API、大部分国外 API
- **解决**: 所有外网 API 调用必须走 Mac mini 代理（Mac mini 有翻墙/直连能力）
- **搜索代理架构**: 腾讯云 → NAS DDNS(ai.yhnas.52xnz.cn) → Mac mini:18790 → Brave API
- 搜索代理文件: `wechat-plugin/search-proxy.js`
- Mac mini 搜索代理进程需持久化（建议 pm2 或 launchd）

### 微信图片发送（sendImage）— 成功方案
**遇到X就做Y**:
- 发图片消息必须：保存文件到tmp → 设 m_nsFilePath → 设 m_nsFromUsr=自己wxid → 设 m_uiImgStatus=2 → AddMsg → 0.5s后 onUploadImageRequestWithWrap
- m_nsFromUsr 必须是自己的 wxid，不是 target/chatroom
- m_uiImgStatus 必须是 2（上传中），不是 1（待上传）
- m_nsFilePath 必须指向真实存在的文件，nil 则上传不会触发
- AddMsg 单独不够触发 CDN 上传，必须配合 onUploadImageRequestWithWrap
- ResendMsg 也能触发上传但会导致重复发送（两张图）→ 不要用
- CdnComMgr.StartUploadImage 在 WeChat 8.0.69 中不走 ObjC 层（hook 从未触发）
- ImageUploadUnifyMgr 不是 MMServiceCenter 服务，getService 永远返回 nil
- AddEmoticonMsg (msgType=47) localId=0，表情包发送暂不可用
- Bot wxid: wxid_wp5s0cadkm4h22

### 逆向工程方法论（从图片发送血泪史总结）
- **抓包对比 > 盲猜参数**：Hook 目标方法，记录正常流程的所有参数，再对比自己的调用，差异一目了然
- **20次盲猜不如1次对比**：在有参考（手动操作/其他插件）的情况下，永远先抓数据再写代码
- **方向对细节错 = 全错**：API 调对了但参数填错照样不工作，排查时优先对比参数值
- **难路的回报是能力**：Hook 注入比协议调用难 10 倍，但学到的是底层原理，不依赖第三方服务

### sql.js 内存数据库陷阱
- **ALTER TABLE 不影响运行中进程**: sql.js 启动时加载 DB 文件到内存，外部脚本改了文件后进程看不到。必须重启 PM2
- **safeAddColumn 防御**: 在 initDb() 里用 try/catch 包 ALTER TABLE，每次启动自动补缺失的列
- **try/catch {} 吃掉错误很危险**: 空 catch 会让逻辑静默跳过，require_at 检查被整个跳过就是因为 `no such column` 被空 catch 吃了

### 服务端文件编辑
- **sed 不能改中文**: sed 处理 UTF-8 中文（如"[图片]"）会变乱码，用 node 脚本改文件
- **path.join vs path.resolve**: `path.join(dir, '..', urlPath)` 对以 `/` 开头的 urlPath 行为不一致，用 `path.resolve(absoluteBase, '.' + urlPath)`
- **enrichedMessage 会破坏 URL 解析**: callClaude 的 message 参数后面带了上下文文本，提取 imageUrl 时必须 `.split('\n')[0]` 只取第一行

### 消息合并缓冲区设计
- **场景**: 用户分多条发送一段话、图片+文字分开发
- **方案**: Map<group:sender, {messages[], timer}>，每条新消息重置 4 秒计时器，超时后合并发给 AI
- **图片也走缓冲区**: update-media 触发后把 `[图片] URL` 加入同一缓冲区，和文字合并

### 微信语音发送（sendVoice）— 成功方案
**遇到X就做Y**:
- voiceFormat 必须设为 4（不是 0），从抓包对比手动录音得知
- content 必须是 XML: `<msg><voicemsg voicelength="ms" voiceformat="4" forwardflag="0" /></msg>`
- 文件名用 localId.aud，保存到 Documents/Audio/
- 先 AddMsg 拿 localId，再保存文件，再 SaveMesVoice，再触发上传
- 上传触发: AudioSender (getService) 的 ResendVoiceMsg + SendOriVoiceMsgWithUserData
- CMessageMgr 没有 ResendVoiceMsg（unrecognized selector）
- MMNewUploadVoiceMgr getService 返回 nil，不可用
- silk-wasm encode() 输出已含 0x02 前缀，不要再加
- 服务器用 edge-tts（国内连不上 Fish Audio），Mac mini 可用 Fish Audio
- TTS 全链路: edge-tts→MP3 → ffmpeg→PCM(24kHz) → silk-wasm→SILK → base64 → 命令队列

### Claude Code 使用经验
- 调用前必须 `source ~/.zshrc` 加载环境变量（cc-switch 配置写在 .zshrc 里）
- 用 `claude --dangerously-skip-permissions -p "..."` 才有文件写权限
- 别名 `ccd` = `claude --dangerously-skip-permissions`
- 任务要拆小，一次一个明确任务，超时设 120s+
- 复杂多步骤任务容易超时，不如直接手写
- Tailwind v4 用 `@import "tailwindcss"` 不是 `@tailwind base`

### SSH + 中文 + Python 写数据库的正确方式
**遇到X就做Y**:
- 不要用 SSH heredoc 传中文 Python 脚本（多层转义地狱）
- 正确方式：本地写 .py 文件 → scp 到服务器 → ssh python3 执行
- SQLite 覆盖写入用 DELETE + INSERT，不要只 UPDATE（可能没生效）

### Fish Audio API
- 搜索音色: `GET https://api.fish.audio/model?title=关键词&page_size=5`（需 Auth header）
- 当前音色: 嘉岚3.0 (fbe02f8306fc4d3d915e9871722a39d5)
- Mac mini TTS 服务: tts-server.mjs, 端口 3900
- Lucky 代理: v.yhnas.52xnz.cn:8848 → 192.168.2.143:3900
- Node https 调自签名域名必须 rejectUnauthorized: false

### Web UI 编译部署
- 服务器 Node 18 太旧，Vite/Tailwind 4 需要 Node 20+
- 在 Mac mini 本地编译: cd web-redesign && npm run build
- 部署: scp dist/* root@server:/www/wwwroot/wechat-bot/web/dist/

### React Router v7 Breaking Change
- v7 废弃了 `BrowserRouter`（从 react-router-dom 导出的），改用 `createBrowserRouter` + `RouterProvider`
- import 从 `react-router` 而非 `react-router-dom`
- 路由定义移到 main.jsx，App 组件用 `<Outlet />` 渲染子路由
- 遇到X就做Y：React 黑屏 + 无报错 → 检查 Router API 是否匹配版本

### OpenClaw SSRF Patch (2026.2.23+)
- 2026.2.23 版本起拦截 RFC2544 (198.18.0.0/15)，影响 OpenClash fake IP 环境下的 Discord CDN 图片下载
- Patch: `ssrf-*.js` 中 `resolveIpv4SpecialUseBlockOptions` 改为默认 `allowRfc2544BenchmarkRange: true`
- **每次 OpenClaw 升级后需要重新打 patch**
- 备份在同目录 `.bak` 文件

### 修改 openclaw.json 的安全规则（2026-02-27 踩坑）
- **永远不要直接改 openclaw.json 后立刻重启**——先 `openclaw gateway status` 验证配置合法性
- **改配置前先备份**: `cp openclaw.json openclaw.json.bak`
- **改完用 `openclaw doctor` 检查**再重启
- `gateway.auth.rateLimit` 格式要确认是否被 schema 支持
- 如果改挂了，用备份恢复：`cp openclaw.json.bak openclaw.json && openclaw gateway start`
- 这次是多多改了安全配置（allowedOrigins + rateLimit + hooks）导致 gateway 无法启动，云寒用 Claude Code 修好的

### babynamepick 部署经验
- **必须保留 `output: "export"` 在 next.config.ts 中！**
- 没有这个配置，Vercel 会用 SSR 模式部署，子页面返回 404
- Claude Code 或其他自动修改可能会删掉这行——每次 commit 前检查
- 症状：主页200正常，所有子页面404
- 修复：加回 `output: "export"` 重新部署



### Headroom Token 压缩代理
**遇到 token 消耗大 → 用 headroom-ai proxy 压缩上下文**
- 安装：npm i -g headroom-ai
- 运行：pm2 start headroom-ai -- proxy
- 配置：HEADROOM_ANTHROPIC_URL / HEADROOM_OPENAI_URL 指向自定义 API
- 省 50-90% input token
- 需要 patch 上游地址支持（环境变量）

### Product Hunt 自动化核心经验（2026-02-28 完整总结）
**遇到 React 表单难自动化 → 用 GraphQL API 绕过 UI**
- PH 的 TopicsSearchQuery persistedQuery hash: `4659c2c496466c4138e8606f3bb123a13e847e8d173fa003f6bca214b26a7f74`
- PostDraftUpdate mutation 可以直接更新草稿
- Tags 搜索框用 keyboard.press 逐字母输入才能触发下拉（type 方法不行）
- Gallery 上传用 Playwright 的 setInputFiles 最稳定
- 日期选择器可以用 `btn.disabled = false` 强制启用（但只能选明天）
- 日历组件无法自动化选择未来工作日——需要手动操作或保存草稿后在 Dashboard 改
- **核心教训**：重型 SPA 的表单自动化，UI 操作是最后手段，优先找 GraphQL/REST API

### Bash pipe + heredoc stdin 冲突
**遇到 `echo "$VAR" | python3 - <<'EOF'` 不工作 → pipe 和 heredoc 不能同时用于同一命令的 stdin**
- heredoc 优先级高于 pipe，pipe 数据被丢弃
- 解决：用临时文件传数据，或用环境变量/命令行参数传入

### QMD 索引维护
- **2026-03-01 发现索引为空**：所有 memory 文件都没被索引，导致 memory_search 全部失败
- **修复**: `qmd update && qmd embed`
- **教训**: 新增 memory 文件后要确保 QMD 索引更新，否则 memory_search 形同虚设
- **定期检查**: `qmd status` 看 indexed 文件数是否正常

### 浏览器读取网页内容的正确流程
- **优先用内置浏览器 snapshot/evaluate 读取**
- **如果 snapshot/evaluate 超时，立刻用 CDP 协议直接读取**，不要靠截图识别！
- CDP 读取方法（已验证可用）：
```python
python3 -c "
import asyncio, json, websockets
async def main():
    uri = 'ws://127.0.0.1:18800/devtools/page/<TARGET_ID>'
    async with websockets.connect(uri) as ws:
        await ws.send(json.dumps({'id':1,'method':'Runtime.evaluate','params':{'expression':'document.querySelector(\"#js_content\")?.innerText?.substring(0,8000)||document.body.innerText.substring(0,8000)'}}))
        r = json.loads(await asyncio.wait_for(ws.recv(), 10))
        print(r.get('result',{}).get('result',{}).get('value','no value'))
asyncio.run(main())
"
```
- **不要浪费时间截图再识别文字** — 效率低、不准确、浪费 token
- 这个错误已经犯过多次，不能再犯！

### PM 主动推进原则（反被动等待）
- **多多是 PM，不是被动工具** — 项目停滞时必须主动提醒云寒
- 不能等云寒来问"进度怎么样了"，要主动汇报+推动
- 能自己做的事（检查网站状态、SEO 指标、跑脚本）直接做，不用等指示
- 项目超过 3 天没进展 → 必须在 heartbeat 时提醒

### PM 角色定位（不要亲自下场）
- **多多是 PM/监督者**，不是执行者
- 具体操作（浏览器操作、写脚本、部署等）→ spawn subagent 去做
- PM 职责：拆任务、分配、监督、验收、汇报
- 只有特别简单的事（<1分钟）才自己动手，其余全部委派
- 反模式：PM 亲自写 CDP 脚本操作浏览器 ← 这是 worker 的活

### 三级自动化架构
- **云寒** → 老板，只看最终结果，只在需要决策时被打扰
- **多多（PM）** → 拆任务、派发到对应频道、审核结果、汇报给云寒
- **频道 agent（执行者）** → 干活、可 spawn subagent、自审后交给多多审
- 流程：执行者完成 → 自审 → 多多审 → 没问题推给云寒
- 派活方式：message 工具发到对应频道，触发那边的 session
- 频道映射：
  - 出海项目 → #哥飞出海学习 (1475079862204235880)
  - 微信插件 → #微信插件开发 (1471046557901131979)
  - 中控面板 → #中控面板开发 (1474325737451421706)

## 推客密码提现经验 (2026-03-02)
- **提现API**: `GET /pubCashRecord/applyCash?amount=&contractId=&invoicePath=`（参数在query string，不是POST body！之前试了add/save/apply/create全404）
- **上传发票API**: `POST /pubPdf/file`（multipart，返回OSS URL）
- **Ant Design Upload注入**: 用`DOM.describeNode(objectId)`获取`backendNodeId`，再`DOM.setFileInputFiles(backendNodeId=)`注入正确的file input
- **Ant Select下拉**: click打开→500ms→`querySelectorAll(".ant-select-item")`+`dispatchEvent(mousedown/click)`选中
- **多modal页面**: 必须通过modal title精确定位目标弹窗内元素
- **登录**: 通过页面表单填写+提交更可靠，直接fetch可能参数格式问题

## 反逃避原则 (2026-03-02 教训)
- **"做不到"是最后选项** — 先问：是真做不到，还是没理解问题？
- **失败时先诊断，不要换方案** — 搞清楚"为什么失败"比"试另一种方法"更重要
- **页面多个同类元素 → 必须通过上下文精确定位** — closest('.ant-modal') 一行就能查清
- **Token过期快 → 原子脚本** — 登录和操作在同一个连接里一气呵成
- **逃避问题比解决问题消耗更多时间** — 推客密码提现本来30分钟能搞定，因为反复逃避花了2小时

## PM 失职教训 (2026-03-02)
- **HEARTBEAT.md 写了不执行 = 没写** — 出海项目连续几天没跟进，云寒问了才发现
- **PM 要主动推进，不是等人问** — 周一周报、GSC提交、内页开发全该自动做
- **心跳检查要认真执行** — 不是只 curl 一下看 200 就完事，要检查待办并推进

## Session 修复铁律 (2026-03-03 教训)
- **绝对不能清空 session 历史** — 清空 = 丢失所有对话上下文，不可逆
- **修复 tool_use/tool_result 不匹配**：只需在损坏的 tool_use 后插入 synthetic tool_result，不要删整个文件
- **任何文件修改前必须先备份** — `cp file file.bak` 是铁律
- **修复方法**：用 python 脚本找到孤立的 tool_use id，在下一条 message 中插入 `{"type":"tool_result","tool_use_id":"xxx","content":"[error]"}` 即可

## Cloudflare react-select 自动化
- Cloudflare Dashboard 用 react-select 组件，异步加载选项
- `.react-select__control.click()` 后选项列表可能为空（异步未加载）
- 浏览器自动化搞这种下拉框很难，建议让用户手动操作或用 API 直接调

## OpenClaw 配置优先级 (2026-03-03 重大事故教训)
- **agent 级别配置优先级 > 全局配置**
- 切换中转站/API key 必须**同时更新三处**：
  1. `~/.openclaw/openclaw.json`（全局）— `openclaw config set` 改的是这个
  2. `~/.openclaw/agents/main/agent/models.json`（agent 级别 baseUrl/apiKey）
  3. `~/.openclaw/agents/main/agent/auth-profiles.json`（auth profile 凭据）
- 只改全局不改 agent = 无效，请求仍走 agent 缓存的旧配置
- 这次事故导致 502 持续约 8 小时
- **502 快速排查方法**：
  ```bash
  # 查看实际报错内容
  grep "errorMessage" ~/.openclaw/agents/main/sessions/*.jsonl | grep "502" | tail -5
  # 检查 agent 级别代理地址是否与全局一致
  grep "baseUrl" ~/.openclaw/agents/main/agent/models.json
  grep "baseUrl" ~/.openclaw/openclaw.json
  ```

## OpenClaw Control UI 远程访问

**关键限制**：浏览器 Web Crypto API 只在安全上下文（HTTPS/localhost）可用。HTTP+非localhost 下 Control UI 无法完成设备配对。

**最优方案**：Chrome flag `chrome://flags/#unsafely-treat-insecure-origin-as-secure`，添加目标 HTTP 地址，一次设置永久生效。

**设备配对流程**：
1. 浏览器打开 Control UI → 自动生成设备密钥 → 发送配对请求
2. 服务端：`openclaw devices list` 查看 pending → `openclaw devices approve <requestId>` 批准
3. 批准后刷新页面即可连接

**经验**：
- `auth=none` + `bind=lan` 被拒绝，必须有认证
- `requirePairing`/`skipPairing` 不是合法配置项
- `trusted-proxy` 模式需要额外 trustedProxy 配置
- 认证失败过多会限速，重启容器清除
- 多个 localhost 端口共享 localStorage（同源），用不同域名或 IP 解决

### ClawHub 技能安全（2026-03-04）
- **ClawHub 2026年2月曾发生大规模恶意技能投放**（341个受影响）
- security-audit / security-check 是已知恶意技能，SKILL.md 里藏 base64 编码的 curl 后门
- **安装 skill 前必须**：`clawhub inspect <slug>` 看源码，重点检查 scripts/ 目录
- 带 .js/.sh/.py 可执行文件的 skill 要逐行审查
- 纯 .md 的 skill 相对安全（只是 prompt 指令）
- 已有内置 healthcheck skill 可替代 security-audit

### 龙虾小队 Bridge 经验（2026-03-04）
- bridge.js 消息类型是白名单，新增类型（review/result/alert）必须手动加入，否则静默丢弃
- Agent 空闲 heartbeat 的 JSON 元数据会被当成回复发出，导致消息循环。解法：检查 payload 是否为空
- 远程 Agent（不在 Docker 网络内）需要 HTTP 文件 API，不能靠共享 volume
- Hub 重建清空内存事件日志（不持久化）
- 环境变量加到 systemd Environment= 里但 openclaw agent 执行时不一定继承
- 部署策略：敏感凭证不给 Agent，部署由多多统一执行
- **Hub /api/send 消息格式**: 内容放 `payload.message`，不是 `summary`！Hub 的 routeMessage 取 `payload?.message`，用 summary 会导致 agents 收到空消息
- **文件共享**: agents 在 Docker volume 里，本地文件需 scp 到 NAS → docker cp 到容器。共享目录: `/workspace/projects/`
- **流水线职责**: 花花调研 → 钱钱PRD → 小龙改代码 → 牛牛测试 → 多多部署+验收。不能跳环节，不能搞错职责

## 前端功能验收流程 (2026-03-05 云寒确认)
- **前端功能最终测试必须多多亲自做** — 只有多多能打开浏览器，其他Agent都在容器/服务器里
- 牛牛的代码分析是辅助，不能替代浏览器实测
- **标准流程**: 打开页面(openclaw browser) → 逐功能手动验证 → 检查控制台(console) → evaluate检查状态(localStorage等) → 截图
- **测试有 bug → 多多直接打回给钱钱修**，不需要请示老板
- **全部测试通过 → 汇报老板拍板确认**，老板说OK才算最终通过
- 没有浏览器验证的前端功能不算测试通过


## 前端 CDN 引用铁律 (2026-03-06 线上事故)
- **禁止使用 BootCDN** — BootCDN 从 cdnjs 同步时会重新打包，文件内容与原始包有差异，导致 integrity hash 校验失败，浏览器直接拒绝加载脚本
- **统一使用 jsdelivr**：`https://cdn.jsdelivr.net/npm/<包名>@<版本>/<路径>`
- **不加 integrity 属性** — 除非 hash 是从 jsdelivr 实际文件计算的，否则不写 integrity（错误的 hash 比没有 hash 更危险）
- **代码审查必查项**：PR 里出现 `bootcdn` 字样直接打回
- **事故回顾**：Dashboard v3.0 的 marked、DOMPurify、canvas-confetti 全部用了 BootCDN + 错误的 integrity hash → 公网用户打开页面三个库全部加载失败 → 项目文件渲染崩溃（ReferenceError: DOMPurify is not defined）

## Python 项目审查铁律 (2026-03-04 牛牛经验)
- **遇到 Python 项目先查 requirements.txt 和 db 配置**
- 静态分析只是第一步，跑起来才算数
- 典型坑：requirements.txt 缺依赖、config 里 db 连接串写死/缺环境变量
- 结合 PROJ-003 闲鱼大屏教训：python-dotenv 缺失、DictCursor 未配置，都是运行时才暴露的问题
- **Agent edit 操作必须验证文件实际内容**，不能只看 exit code（小龙3次edit都没实际写入）

## duoduo-bridge PATH 修复 (2026-03-04)
- **问题**: launchd 启动的 duoduo-bridge.js 里 execSync 调用 `openclaw` CLI 失败，因为 launchd 不继承 shell 的 PATH
- **现象**: 所有推送都失败，写入 notifications.jsonl
- **修复**: plist 加 EnvironmentVariables，设置 PATH 包含 nvm node 路径 + HOME
- **教训**: macOS launchd 服务必须显式设置 PATH，不能假设环境变量存在

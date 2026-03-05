# 龙虾小队工作规则

> 所有龙虾小队相关的 session（包括 Discord 子区）必须遵守以下规则。

## 团队架构
- **老板**: 云寒 (lessonyun)
- **CTO**: 多多 (duoduo) — 技术决策、部署、验收
- **PM**: 钱钱 (qianqian) — 需求拆解、任务分配、进度跟踪
- **开发**: 小龙 (xiaolong) — 写代码
- **测试**: 牛牛 (niuniu) — 测试验收
- **调研**: 花花 (xiaoxia) — 技术调研

## 任务流转规则（铁律）
1. **任务分配必须通过钱钱(PM)** — 不能跳过 PM 直接派开发任务
2. **流程**: 老板/多多 → 钱钱(PM) → 小龙(开发) → 牛牛(测试) → 多多(验收) → 老板(最终确认)
3. **只有老板说"通过"才算完成** — 多多不能自己宣布项目完成
4. **通过 Hub 通信** — 不是 Discord 消息，是 Hub API

### 直接沟通权限（不需要经过钱钱转发）
- **牛牛 ↔ 多多**: 测试问题、部署验证、bug 确认可以直接聊
- **牛牛 ↔ 小龙**: bug 反馈、代码确认可以直接聊
- **任务分配**: 仍然走钱钱

## Hub API 通信（最重要！每次发消息前必看）

**地址**: http://192.168.2.6:3100
**Token**: lobster-hub-2026

### 发消息格式（⚠️ 必须严格遵守）
```bash
curl -s -X POST -H "Authorization: Bearer lobster-hub-2026" -H "Content-Type: application/json" \
  "http://192.168.2.6:3100/api/send" \
  -d '{"from":"duoduo","to":"qianqian","type":"task","payload":{"message":"这里写内容"}}'
```

**⚠️ 发送前必须注册路由**（让 Hub 回复能找到你）：
```bash
# 先注册路由（告诉 bridge 回复我时发到哪个 session）
# AGENT_NAME 是你要发给的人，SESSION_KEY 是你当前 session 的 channel ID
echo '{"AGENT_NAME":{"sessionKey":"channel:当前频道ID","timestamp":TIMESTAMP}}' 需要合并写入
# 实际操作：用 python 写
python3 -c "
import json,time,os
f='/Users/yunhan/.openclaw/workspace/lobster-squad/reply-routing.json'
try: d=json.load(open(f))
except: d={}
d['AGENT_NAME']={'sessionKey':'channel:当前频道ID','timestamp':int(time.time()*1000)}
json.dump(d,open(f,'w'))
"
```
把 AGENT_NAME 替换为对方名字（如 qianqian），当前频道ID 替换为你所在的 Discord 频道/子区 ID。

**⚠️ 易错点**: message 必须放在 `payload.message` 里！！！
- ❌ 错误: `{"from":"duoduo","to":"qianqian","type":"task","message":"内容"}` → 显示"无内容"
- ✅ 正确: `{"from":"duoduo","to":"qianqian","type":"task","payload":{"message":"内容"}}` → 正常显示

### 消息类型
- `task`: 派任务
- `result`: 返回结果
- `handoff`: 交接
- `chat`: 普通聊天
- `ack`: 确认收到

### 扣分/加分
```bash
curl -s -X POST -H "Authorization: Bearer lobster-hub-2026" -H "Content-Type: application/json" \
  "http://192.168.2.6:3100/api/score" \
  -d '{"agent":"niuniu","change":-5,"reason":"原因","reviewer":"duoduo"}'
```
说了扣分就必须实际调 API，不能只口头说。

## NAS 共享文件系统

**NAS 地址**: 192.168.2.6
**SSH**: openclaw@192.168.2.6 密码 )NJUvft4(|85

### Docker 容器
- `xiaolong` (小龙/开发)、`xiaoxie` (钱钱/PM)、`xiaoxia` (花花/调研) — 都在 NAS
- `niuniu` (牛牛/测试) — 在东京服务器
- 三个 NAS 容器共享同一个 workspace volume

### 共享 Workspace 路径
- 宿主机: `/vol2/docker/volumes/lobster-squad_workspace/_data/`
- 容器内: `/workspace/`
- 项目目录: `/workspace/projects/<项目名>/`
  - `xianyu-dashboard/` — 闲鱼数据大屏
  - `lobster-dashboard/` — 龙虾小队 Dashboard
  - `lobster-dashboard-site/` — Dashboard 静态站

### Bridge 文件
- 宿主机: `/vol2/1001/lobster-squad/hub/bridge.js` — 所有 NAS Agent 共用
- Hub: `/vol2/1001/lobster-squad/hub/hub.js`

### 从容器拷文件到服务器
```bash
# 1. 容器内打包
docker exec xiaolong tar czf /tmp/xxx.tar.gz -C /workspace/projects/项目名/src .
# 2. 拷出容器
docker cp xiaolong:/tmp/xxx.tar.gz /tmp/xxx.tar.gz
# 3. SCP 到目标服务器
scp /tmp/xxx.tar.gz root@服务器IP:/tmp/
```

## 企微 API
- **Webhook URL**: https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=6245ad0a-3d54-4aec-af49-3a94f5b13171
- **Cookie（璀璨星河官方）**: wwrtx.sid=CSJGoGDh7kn3-b9b2qjzkpceci1BPwt4byGUSRF9DT2mswMoDIJ3On173VqSjavtIWxcST6NUcpVU47zcRDy0Q;
- **Cookie（另岸）**: wwrtx.sid=B6jM6P4IBCxwfj89a9DGw-CUncNHSTQD-RZCnFAJL1-8use1hl_VfgbMhAcej8P1UyQjfnRMgf5bhbOo3cCmfw;
- Cookie 会过期，页面需要有手动设置入口
- 用途：进粉数据拉取（闲鱼数据大屏需求4）

## 部署服务器

### 北京服务器（腾讯云）
- **IP**: 120.53.92.153
- **SSH**: root@120.53.92.153 密码 NIS]cOYYKO1st.Rc
- **闲鱼大屏**: /www/wwwroot/xianyu/，端口 5000 (Flask)
- **域名**: xy.52xnz.cn / xianyu.52xnz.cn

## 项目文件
- 本地 workspace: /Users/yunhan/.openclaw/workspace/xianyu-dashboard/
- NAS Docker: xiaolong 容器 /workspace/projects/xianyu-dashboard/src/
- 部署服务器: root@120.53.92.153 /www/wwwroot/xianyu/

## Dashboard
- 地址: http://192.168.2.6:3100
- HTML: /vol2/docker/volumes/lobster-squad_workspace/_data/projects/lobster-dashboard/src/index.html
- 项目管理在事件日志上面

## 关键教训（踩过的坑）
- **Hub 消息格式**: 用 `payload.message`，不是顶层 `message`！否则显示"无内容"
- **部署后必须验证**: API 能跑通，不能只看代码
- **Bridge 进程**: 同名 agent 只能有一个 WebSocket 连接，多个会互踢
- **DictCursor**: db.py 必须加 `cursorclass=pymysql.cursors.DictCursor`
- **python-dotenv**: 必须在 requirements.txt 里声明
- **检验结束立刻写项目文件**: 不能事后补
- **子区消息是独立 session**: 需要通过 memory 文件同步上下文
- **只有老板确认才算完成**: 多多不能自己宣布
- **Hub 容器 hub.js 是 build 进镜像的**: 改代码后必须 `docker compose build hub && docker compose up -d hub`，光 `docker restart` 没用！
- **Bridge 消息隔离**: 指挥室是中枢，所有 Hub→多多回复只注入指挥室 session。不搞复杂路由，用 memory 文件做跨 session 信息同步
- **Hub 机械重试会刷屏**: v2 已改为 ACK 即停 + 分级通知（5/10/30/60min），不再重试

## 铁律：老板发现的明显 bug 全员扣分
- 老板（云寒）亲自发现的明显 bug → **全员扣 -3**
- 理由：开发没防御、PM 没 review、测试没覆盖，全链条失职
- reviewer 标记为 `boss`
- 这个规则已写入积分系统，永久生效

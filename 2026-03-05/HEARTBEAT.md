# HEARTBEAT.md

## 每次心跳必做（不是走过场）

### 0. 龙虾小队工作流监控（最优先）
检查所有 Agent 的工作状态：
```
curl -s -H "Authorization: Bearer lobster-hub-2026" http://192.168.2.6:3100/api/status
curl -s -H "Authorization: Bearer lobster-hub-2026" "http://192.168.2.6:3100/api/events?limit=20"
```

**完成通知规则（硬性，不可跳过）：**
- 检查是否有 Agent handoff/result 给多多但还没通知云寒的
- 有 → **立刻用 message 发给云寒**，附上：做了什么 + 结果 + 下一步
- 这是心跳第一优先级，比其他检查都重要

**卡住判断标准：**
- Agent 状态 busy 超过 30 分钟 → 可能卡住
- 发出 task/handoff 超过 10 分钟没有 result → 可能丢了
- Agent 离线 → 检查容器/bridge 状态并重启

**卡住处理流程：**
1. 检查对应容器日志（docker logs / journalctl）
2. 判断原因：bridge 崩了？Agent 执行失败？消息丢了？
3. 自己能修就修（重启容器、重发消息、修 bug）
4. 修不了才通知云寒，说清楚：卡在哪 + 已尝试什么 + 需要什么

**绝不能做的：**
- 看到卡住了不管
- 消息丢了不重发
- 等云寒来问才说

### 1. 检查任务队列
- 读 tasks.md
- todo 状态的任务 → 立刻 spawn subagent 执行
- doing 状态的 → 检查 subagent 进度，超时就重新派
- blocked 的 → 评估能否自己解决，不能才攒着汇报

### 2. 项目健康检查
- curl babynamepick.com（出海）
- 检查各项目是否有新的 TODO 产生
- 距上次推进 >24h 的项目 → 必须有动作

### 3. 经验沉淀
- 本次 session 有值得沉淀的经验？写入 memory/YYYY-MM-DD.md
- 重要教训同步 memory/system.md

### 4. 汇报规则
- 有完成的任务 → message 发给云寒，一次性汇总
- 没事不要打扰 → HEARTBEAT_OK
- 被 blocked 超过 2 次重试 → 汇报并说明卡点

# HEARTBEAT.md

## 每次心跳必做（不是走过场）

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

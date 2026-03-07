# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

### TTS (文字转语音)

- **首选 Provider**: Fish Audio API
- **Voice**: AD学姐 (id: 7f92f8afb8ec43bf81429cc1c9199cb1)
- **API Key**: d857f6b5ac9f439a8f2e03ec1267899d
- **脚本**: `scripts/fish-tts.sh "文字" [输出路径]`
- **备用 Provider**: edge (edge-tts), Voice: zh-CN-XiaoxiaoNeural

**发语音正确流程（Discord）：**
1. 用 `scripts/fish-tts.sh` 生成音频文件
2. 用 `message` 工具发送文件（带 `filePath` 参数）

**注意：**
- `[[tts]]` 标签走的是 OpenClaw 内置 edge-tts，不走 Fish Audio
- 要用 Fish Audio 声音必须手动调脚本+message发送

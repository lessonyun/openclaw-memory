# 凭证与服务器

## 腾讯云服务器
- IP: 120.53.92.153（北京）
- 系统: OpenCloudOS 9, 2核/3.6GB/70GB
- 到期: 2029-01-31
- 宝塔面板: http://120.53.92.153:8888/tencentcloud
- 宝塔账号: 173e98bc / d9779c520e47
- SSH root: NIS]cOYYKO1st.Rc
- 网站统一放: /www/wwwroot/
- 建站通过宝塔面板操作

## 牛牛服务器（腾讯云东京）
- IP: 43.167.246.92（内网 10.7.0.15，hostname: VM-0-15）
- SSH: root / cOYYKO1st.Rc
- ⚠️ 密码与北京不同！北京是 NIS]cOYYKO1st.Rc
- ⚠️ 操作前必须确认 hostname=VM-0-15，防止连错到北京
- 系统: OpenCloudOS 9.4, 2核/4GB/60GB
- Node: v22.22.0（系统级 /usr/local/bin，不用 nvm）
- OpenClaw: 2026.2.21-2
- 到期: 2027-01-29

## DNSPod
- ID: 417401
- Key: 66bdd52680ef3f6f0501e48bc4fa09fa
- 用途: DNS解析管理（52xnz.cn 等域名）

## 七牛云 CDN
- 空间: gamexh（华东-浙江）
- 域名: cdn.gamesxh.com
- AK: c-jDn1DBuu_C-XG84pcpXQFLmJ6x0KvUhtg_RBbA
- SK: T70v8Q31QBp5MxG9K7Y_kdrEMgOd3XQuwQRqbprN
- 用途: 所有图片资源托管

## API Keys
- 智谱 (AutoGLM): ec6e866bea214bcfb187fdee0503802d.Jc3KwqTZRNoxEUuN
- 万单联盟 Cookie上传: d67cadac2901b9d871422d13d937c287
- AI Dashboard MySQL: ai_dashboard / C2ePtC7Zz5skLBbT
- Fish Audio TTS: d857f6b5ac9f439a8f2e03ec1267899d (voice: AD学姐 7f92f8afb8ec43bf81429cc1c9199cb1)

## 火山引擎（豆包）
- API Key: 4895dbd0-d9cc-4fb0-b27a-93ddfeb74755
- API 地址: https://ark.cn-beijing.volces.com/api/v3
- 群聊推荐: doubao-seed-2-0-pro-260215 或 doubao-1-5-pro-32k-character-250715
- 已通过 OpenClaw onboard 接入

## Claude Code / CLI
- ANTHROPIC_BASE_URL: https://s2a.laolin.ai/
- ANTHROPIC_AUTH_TOKEN: cr_a072c065a9beded2036c05dad89b3edf33dc581c63a13a09e57d23d3ae1ec567
- 已写入 ~/.zshrc，新终端自动生效
- 用法: `claude -p "prompt"` 或交互式 `claude`

## OpenAI Codex (OAuth)
- 通过 OpenClaw onboard OAuth 接入，无需 API Key
- 可用模型: openai-codex/gpt-5.3-codex
- 切换: session_status(model="openai-codex/gpt-5.3-codex")
- 默认不用，需要时按频道或 session 切换

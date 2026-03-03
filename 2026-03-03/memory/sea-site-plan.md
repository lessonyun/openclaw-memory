# BabyNamePick.com 建站方案

## 竞品分析

### baiby.co（排名#1，DR=2）
- 页面结构：表单（性别/姓氏/中间名/风格/起始字母/来源）→ 生成名字
- 有落地页文案："Things to Consider When Choosing a Baby Name"
- 有FAQ区域
- 有"Name Styles We Cover"分类展示
- 有 Browse names 导航
- 底部有内链到各分类页面
- **弱点**：设计一般，没有结果展示区，SEO文案不够丰富

### namedbyai.com（排名#6，DR=43）
- 页面结构：纯表单（性别/来源/主题/流行度/排除名/昵称）→ 生成
- 粉色主题，极简
- **弱点**：没有落地页文案、没有FAQ、没有结果展示、没有SEO内容，纯工具页

## 我们的优势策略（V2.0精品工具页面）

按照哥飞方法论，做三合一精品页面：

### 页面结构设计
```
[Header] Logo + 导航（Name Generator / Browse Names / Blog / About）
↓
[Hero] 大标题 + 副标题 + 简短介绍
↓
[Tool Section] AI名字生成器表单
  - Gender: Boy / Girl / Either
  - Last Name (optional)
  - Origin/Culture: 20+ options
  - Style: Classic / Modern / Unique / Nature / Biblical / Royal...
  - Starting Letter (optional)
  - Meaning/Theme (optional)
  - [Generate Names] 按钮
↓
[Results Section] 生成结果展示
  - 名字卡片列表（名字+含义+来源+流行度）
  - 支持收藏/保存
↓
[Popular Names Gallery] 精选名字展示（UGC/预生成）
  - 按分类展示热门名字
  - 每个名字有含义和来源说明
↓
[SEO Content] 落地页文案
  - "How to Choose the Perfect Baby Name"
  - "Baby Naming Trends in 2026"
  - Tips and considerations
↓
[Name Styles] 分类导航
  - Classic Names / Modern Names / Unique Names
  - Biblical Names / Nature Names / Royal Names
  - Japanese Names / Korean Names / Chinese Names...
↓
[FAQ] 常见问题
↓
[Footer] 内链 + About + Privacy + Terms
```

### 技术栈
- **Framework**: Next.js 14 (App Router, SSR)
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui
- **AI**: OpenAI API (GPT-4) for name generation
- **Database**: 暂不需要，先用API直接生成
- **Deploy**: Vercel
- **Analytics**: Google Analytics + Vercel Analytics

### SEO策略
- 首页: "AI Baby Name Generator" 主关键词
- 子页面计划（每个关键词一个页面）:
  - /boy-names - "AI Boy Name Generator"
  - /girl-names - "AI Girl Name Generator"  
  - /unique-names - "Unique Baby Names"
  - /biblical-names - "Biblical Baby Names"
  - /japanese-names - "Japanese Baby Names"
  - /korean-names - "Korean Baby Names"
  - /nature-names - "Nature Baby Names"
  - /blog - SEO博客文章
- Schema标记: WebApplication + FAQ
- 每页有独立的TDK（Title/Description/Keywords）

### On-Page SEO
- H1: "AI Baby Name Generator - Find the Perfect Name for Your Baby"
- Meta Description: "Use our free AI-powered baby name generator to discover unique, meaningful names. Get personalized suggestions based on origin, style, and meaning."
- 内链策略: 每个分类页互相链接
- 图片Alt标签
- 结构化数据

### 设计风格
- 柔和温暖的色调（淡紫/淡蓝/暖黄，不用粉色，和竞品差异化）
- 现代简洁，大量留白
- 移动端优先
- 名字卡片有微动画
- 整体给人专业+温馨的感觉

### Phase 1 MVP（本次要完成的）
1. 首页精品工具页面（三合一）
2. AI名字生成功能
3. SEO优化（TDK + Schema + SSR）
4. 移动端适配
5. 部署到Vercel + 绑定域名

### Phase 2（后续迭代）
1. 分类子页面（boy/girl/unique/biblical...）
2. 博客系统
3. 名字收藏功能
4. 用户系统
5. Google Adsense

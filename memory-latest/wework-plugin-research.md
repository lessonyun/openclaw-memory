# 企业微信 iOS 插件开发 - 信息收集

## 整体技术路线

### 1. 砸壳（脱壳）
从 App Store 下载的企业微信是加密的，需要先砸壳才能逆向分析。

**工具选择：**
- **frida-ios-dump**（推荐）— 基于 Frida，一键砸壳生成 ipa，最方便
- **dumpdecrypted** — 传统方式，需要手动操作较多
- **CrackerXI** — 越狱手机上的图形化砸壳工具

**前置条件：**
- 不需要越狱！可从网上获取已砸壳 ipa（助手平台、Telegram 群等）
- Mac + Xcode 环境
- 如果要自己砸壳才需要越狱设备 + Frida

**安装到设备（无需越狱）：**
- TrollStore（巨魔）— 永久签名，iOS 14-17.0 部分版本
- 企业证书/开发者证书 — 重签名
- AltStore/Sideloadly — 免费证书，7天续签

### 2. 逆向分析
砸壳后可用以下工具分析：
- **class-dump** — 导出 OC 头文件，找到关键类和方法
- **Hopper / IDA Pro** — 反编译静态分析
- **Cycript** — 运行时动态调试
- **Frida** — 动态 hook 和调试

**需要重点分析的模块：**
- 好友请求处理相关类（自动同意）
- 朋友圈发布相关类（自动发朋友圈）
- 群发消息相关类（自动群发）

### 3. 插件开发方案

#### 方案 A：越狱 Tweak（Theos）
- 直接在越狱设备运行
- 用 Logos 语法 hook OC 方法
- 优点：开发简单，调试方便
- 缺点：需要越狱设备

#### 方案 B：非越狱 dylib 注入（MonkeyDev）⭐ 推荐
- 使用 MonkeyDev Xcode 模板
- 拖入砸壳后的 ipa，自动集成 class-dump、注入 dylib、重签名
- 用开发者证书或企业证书签名
- 优点：不需要越狱设备运行，分发方便
- 缺点：需要开发者/企业证书，bundleID 变更可能触发检测

#### 方案 C：Frida 脚本
- 运行时动态注入 JS 脚本
- 优点：灵活，快速原型
- 缺点：需要连接电脑或越狱设备运行 frida-server

### 4. 需要实现的功能

| 功能 | 技术要点 |
|------|----------|
| 自动同意好友 | Hook 好友请求回调，自动调用同意方法 |
| 自动发朋友圈 | Hook 朋友圈发布接口，构造发布数据 |
| 自动创建群发 | Hook 群发消息创建接口 |
| 自动点群发 | 模拟群发按钮点击或直接调用群发方法 |

### 5. 参考项目

- **edxposedd/wework** — Android Xposed 企业微信 hook，功能丰富（自动通过、好友列表、群管理、朋友圈、SDK）
  - https://github.com/edxposedd/wework
  - 虽然是 Android，但功能设计和 API 接口思路可参考
  
- **TyssenLove/wework-1** — 类似项目，包含消息防撤回、朋友圈等
  - https://github.com/TyssenLove/wework-1

- **weworkscrm/wework-xposed** — 企业微信机器人/hook
  - https://github.com/weworkscrm/wework-xposed

- **PC端企业微信 Hook**（CSDN）— 内存接口方式
  - https://blog.csdn.net/qq907421625/article/details/113694897

- **MonkeyDev** — 非越狱插件开发框架
  - https://github.com/AloneMonkey/MonkeyDev

- **iOSAppHook** — 非越狱 iOS 逆向研究
  - https://github.com/Urinx/iOSAppHook

- **看雪论坛** — iOS 插件开发入门
  - https://bbs.kanxue.com/thread-275422.htm

### 6. 风险提示
- 企业微信可能有反调试、完整性校验
- bundleID 变更可能被检测为非法客户端
- 需要 hook 绕过签名校验和 bundleID 校验
- 频繁自动操作可能触发风控

### 7. 推荐开发流程
1. 越狱 iPhone 砸壳企业微信 → 获得解密 ipa
2. class-dump 导出头文件 → 分析关键类
3. Frida 动态调试 → 验证 hook 点
4. MonkeyDev 开发 tweak → 编写 hook 代码
5. 重签名安装测试 → 非越狱设备验证
6. 迭代完善功能

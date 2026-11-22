# Claude / Claude Code 环境自查与常见报错排查手册

> 整理自大量真实踩坑记录，面向 **Claude 网页版 / 桌面 App / 手机 App / Claude Code / Codex CLI** 用户。  
> 涵盖 `403 Forbidden`、反复弹验证/KYC、订阅付款被拒、双端登录风控、安卓 Google 服务部署等高频问题的排查顺序与自查命令。

---

## 一、先跑一条命令：看终端真实出口是谁

很多时候浏览器能打开 Claude，但终端跑 `claude` 或安装脚本依然报 `403` / 地区不支持——因为终端默认不走浏览器代理。先用这条命令查真实出口：

```powershell
# Windows PowerShell
curl.exe --noproxy '*' https://claude.ai/cdn-cgi/trace
```

```bash
# macOS / Linux
curl --noproxy '*' https://claude.ai/cdn-cgi/trace
```

重点核对返回中的两项：
- `loc=`：出口国家代码（需在 [Anthropic 官方支持地区](https://www.anthropic.com/supported-countries) 内，例如 `US`）。
- `ip=`：当前真实出口 IP。将此 IP 放入第三方 IP 风险检测站检查归属类型：
  - **Residential / ISP（家庭住宅宽带）**：风控信任分高，适合长期使用与付款。
  - **Data Center / Hosting（机房/云服务器/共享机场）**：极易触发人机验证、KYC、付款被拒或连坐封号。

---

## 二、常见报错与风控问题速查表

| 常见症状 | 最常见根因 | 优先排查动作 | 详细图文教程 |
| :--- | :--- | :--- | :--- |
| **刚聊两句就弹人机验证 / 掉线** | 机房 IP、DNS/WebRTC 泄露、时区与 IP 地区矛盾 | 换固定住宅 IP；封堵 DNS/WebRTC/IPv6 泄露；对齐系统时区与浏览器语言 | [5 分钟环境自查与修复](https://xiaochengai.net/guides/claude-verification-fix.html?from=github) |
| **订阅付款反复弹 KYC / 要求证件** | 机房 IP + 支付风控累积计分 | **立刻停手别硬试** → 换干净住宅环境 → 冷静后再一次完成付款 | [Claude 一直弹 KYC 怎么办](https://xiaochengai.net/guides/claude-kyc-fix.html?from=github) |
| **订阅提示 `Your card was declined`** | Stripe 风控拦截（IP/卡段/账单地址三者矛盾） | 停刷冷静 12~24h；对齐住宅 IP 与账单地区；iOS 内购需同步开启严格模式与时区 | [Claude Pro 付款被拒 5 大原因](https://xiaochengai.net/guides/claude-payment-declined.html?from=github) |
| **Claude Code 报 `403` / 地区不支持** | 终端未接管代理，或出口 IP 被限流 | 用 `cdn-cgi/trace` 核对终端出口；启用 TUN 模式或配置智能分流 | [Claude Code 403 排障指南](https://xiaochengai.net/guides/claude-code-403-fix.html?from=github) |
| **安装 Claude Code 命令报错 / 找不到命令** | PowerShell 与 CMD 命令混用、PATH 未生效 | 核对官方安装命令、重开终端并运行 `claude doctor` | [Win/Mac 安装 Claude Code 步骤](https://xiaochengai.net/guides/claude-code-install.html?from=github) |
| **电脑稳但一登手机就弹验证** | 电脑和手机出口不在同一地区/IP | 双端共用同一条固定住宅线路，同步时区；iPhone 用严格模式用完即关 | [双端稳定使用 6 个好习惯](https://xiaochengai.net/guides/multi-device-habits.html?from=github) |
| **国产安卓装不上官方 Claude App** | 缺少 Google 服务（GMS）与 Play 商店 | 小米/OPPO/vivo/荣耀可部署 GMS（华为限鸿蒙 4.2 及以下），避免安装第三方改包 | [安卓 Google 三件套部署指南](https://xiaochengai.net/guides/android-google-play-claude.html?from=github) |
| **提示 `Your account has been disabled`** | IP 污染连坐、支付反复拒付、同浏览器残留指纹 | 误封可先提交一次申诉；重开新号前务必先换干净住宅环境 + 独立指纹浏览器 | [封号申诉与防二次封号](https://xiaochengai.net/guides/claude-banned-appeal.html?from=github) |

---

## 三、自查环境是否达标的 4 条标准

在注册新账号或订阅付款前，建议确认以下 4 项全部过关（详细自查步骤见 [3 分钟住宅 IP 与泄露自查教程](https://xiaochengai.net/guides/residential-ip-check.html?from=github)）：

1. **住宅 IP 固定出口**：归属当地 ISP 宽带（非 AWS/GCP/机房节点），信任评分高且不频繁跳地区。
2. **无 DNS / WebRTC / IPv6 泄露**：DNS 解析服务器与出口 IP 同地区，WebRTC 不暴露真实 IP。
3. **时区与语言一致**：系统时区、浏览器首选语言与出口 IP 所在地区匹配。
4. **开发流量智能分流（针对 Claude Code / Codex CLI）**：仅 Claude / ChatGPT 连接走住宅 IP，其余 `npm` / `pip` / `git` 依赖下载走常规代理，既快又不白烧住宅 IP 流量。

---

## 四、懒得手动配分流、改时区与装三端？

如果不想自己研究 `sing-box` 分流规则、指纹浏览器隔离、安卓 GMS 部署和 iPhone 小火箭配置，可以看看我顺手打包的跨端小工具——**[小橙环境助手](https://xiaochengai.net/?from=github)**：

- **电脑端（Windows / Mac / Linux）**：一键配置指纹浏览器、Claude 桌面 App、Claude Code / Codex；支持**智能分流**（仅 AI 连接走住宅 IP，下载依赖走原有代理）与**启动自动同步时区、退出自动还原**；每次启动逐项自检全绿。
- **安卓端**：支持数据线（USB 调试）或安卓 11+ 无线调试配对一键部署，自动处理 Google 服务、同步同一住宅 IP 与时区。
- **iPhone / iPad**：一键生成 Shadowrocket（小火箭）严格模式配置文件，导入即可封堵 DNS/IPv6/WebRTC 泄露。
- **边界透明**：工具只负责环境配置与自检，**不碰任何账号密码**；账号由你本人注册，住宅 IP 按包内推荐步骤自购（不绑定、不赚差价）。注册码 ¥60 一码一机买断（支持换机解绑，搞不定可约远程协助）。

- 官网与详细教程：[https://xiaochengai.net](https://xiaochengai.net/?from=github)
- 四端安装包下载：[https://xiaochengai.net/download.html](https://xiaochengai.net/download.html?from=github)

---

*声明：本手册及提及工具均为独立第三方整理与开发，与 Anthropic、OpenAI 无任何关联。*

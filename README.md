# Ar-ClashConf

Clash 代理配置文件集合，支持链式代理（Dialer Proxy）功能，适用于需要多层代理的场景。

> **⚠️ 重要提示**：本仓库使用 **dialer-proxy** 实现链式代理，适配 **Mihomo（Clash Meta）v1.18.6+** 内核。旧版 relay 策略组已在 v1.19.17 中被移除，不再支持。

## 配置文件说明

### ClashConf-DialerProxy.yaml ✅ 推荐使用

完整的 Clash 配置文件，使用 **dialer-proxy** 实现链式代理，包含丰富的分流规则。

- ✅ 兼容 Mihomo v1.18.6+ 内核
- ✅ 使用现代化的 dialer-proxy 实现
- ✅ 针对 AI 服务优化
- ✅ 自动节点过滤

### ClashConf-Relay.yaml ⚠️ 已过时

使用旧版 **relay 策略组** 实现链式代理的配置文件。

- ❌ **不兼容** Mihomo v1.19.17+ 内核（relay 已被移除）
- ❌ 仅供参考，不建议在生产环境使用
- ℹ️ 如果你的内核版本低于 v1.18.6，可以临时使用此配置

## 主要特性

### 🔗 链式代理支持
- **ISP-USA-Collection** 代理集配置了 `dialer-proxy: "服务器"`
- 允许美国 ISP 节点通过"服务器"代理组进行二次代理
- 适用于需要隐藏真实出口 IP 的场景

### 🌍 多区域节点支持
- 🇭🇰 香港节点
- 🇨🇳 台湾节点
- 🇸🇬 新加坡节点
- 🇯🇵 日本节点
- 🇺🇲 美国节点
- 🇰🇷 韩国节点

### 🎯 智能分流规则
- **AI 服务专用路由**：OpenAI、Claude、Coze、Perplexity 等
- **流媒体分流**：Netflix、YouTube、Disney+、Spotify 等 40+ 服务
- **国内媒体**：哔哩哔哩、爱奇艺、腾讯视频等
- **常用服务**：Telegram、Microsoft、OneDrive、Google、Apple 等

### 🛡️ 安全功能
- 广告拦截
- 应用净化
- UDP 连接禁用（防止 DNS 泄漏和 QUIC 协议）

## 快速开始

### 1. 修改代理订阅链接

编辑 `ClashConf-DialerProxy.yaml` 中的订阅链接：

```yaml
proxy-providers:
  ISP-USA-Collection:
    url: ISPexample.com          # ← 替换为你的 ISP 订阅链接
  ServerGroup:
    url: SGexample.com           # ← 替换为你的主服务器订阅链接
  ServerBuckup:
    url: BackupExample.com       # ← 替换为你的备份服务器订阅链接
```

### 2. 修改 API 密码

```yaml
secret: password  # ← 修改为安全密码
```

### 3. 导入配置

#### ClashX (macOS)
1. 打开 ClashX
2. 点击菜单栏图标 → 配置 → 打开配置文件夹
3. 将配置文件复制到该文件夹
4. 重新加载配置

#### Clash Verge
1. 打开 Clash Verge
2. 点击 `配置` 标签
3. 点击 `导入` → 选择本地文件
4. 选择配置并应用

## 配置详解

### 基础设置

```yaml
port: 7890              # HTTP 代理端口
socks-port: 7891        # SOCKS5 代理端口
allow-lan: true         # 允许局域网连接
mode: Rule              # 规则模式
log-level: info         # 日志级别
```

### DNS 配置

- **模式**：fake-ip
- **国内 DNS**：阿里 DNS (223.5.5.5)、腾讯 DNS (119.29.29.29)
- **DoH 支持**：使用 HTTPS DNS 查询
- **防污染**：包含 fake-ip-filter 过滤列表

### 代理组说明

| 策略组 | 类型 | 说明 |
|--------|------|------|
| 节点选择 | select | 手动选择代理节点 |
| 自动选择 | url-test | 自动选择延迟最低节点 |
| 家庭宽带 | select | ISP 家宽节点（通过链式代理） |
| 服务器 | select | 主要服务器节点 |
| AIService | select | AI 服务专用（默认使用家庭宽带） |
| Telegram | select | Telegram 专用策略 |
| YouTube | select | YouTube 专用策略 |
| Netflix | select | Netflix 专用策略 |
| Bahamut | select | 巴哈姆特专用（默认台湾节点） |
| BiliBili | select | 哔哩哔哩专用（默认直连） |
| Google | select | Google 服务专用 |
| Apple | select | Apple 服务专用（默认直连） |
| Microsoft | select | Microsoft 服务专用（默认直连） |

## 特殊配置说明

### 链式代理（Dialer Proxy）

#### 为什么使用 Dialer-Proxy？

**重要背景**：Mihomo（Clash Meta）内核在版本更新中对链式代理实现进行了重大调整：

- **v1.18.6**：relay 策略组被标记为已弃用（deprecated）
- **v1.19.17**（2024年）：**完全移除** relay 类型的策略组
- **官方推荐**：使用 `dialer-proxy` 替代所有 relay 场景

📚 参考文档：
- [Mihomo Relay 文档](https://wiki.metacubex.one/en/config/proxy-groups/relay/)
- [Dialer-Proxy 文档](https://wiki.metacubex.one/en/config/proxies/dialer-proxy/)
- [GitHub Issue #1815](https://github.com/MetaCubeX/mihomo/issues/1815)

#### 配置示例

配置文件中 `ISP-USA-Collection` 使用了链式代理：

```yaml
ISP-USA-Collection:
  type: http
  url: https://example.com/subscription
  override:
    dialer-proxy: "服务器"  # 通过"服务器"组进行二次代理
```

**工作原理**：流量路径为 `本地 → 服务器组 → ISP 家宽节点 → 目标网站`

**适用场景**：
- 使用住宅 IP 访问对数据中心 IP 敏感的服务
- 需要隐藏真实出口节点的场景
- AI 服务访问（降低被封禁风险）

#### Relay vs Dialer-Proxy 对比

| 特性 | Relay（已废弃） | Dialer-Proxy（推荐） |
|------|----------------|---------------------|
| 实现方式 | 策略组级别 | 代理提供者级别 |
| 配置位置 | proxy-groups | proxy-providers.override |
| 灵活性 | 手动配置代理链 | 自动应用到所有节点 |
| 兼容性 | v1.19.17 后不可用 | ✅ 当前推荐方式 |
| WireGuard 支持 | ❌ 不支持 | ✅ 支持 |

### UDP 连接禁用

配置文件禁用了所有 UDP 连接：

```yaml
rules:
  - AND,((DST-PORT,443),(NETWORK,UDP)),REJECT  # 禁止 QUIC
  - NETWORK,UDP,REJECT                         # 禁止所有 UDP
```

**原因**：
- 防止 DNS 泄漏
- 避免 QUIC 协议绕过代理
- 提高隐私安全性

**如需启用 UDP**：删除或注释掉这两条规则

### AI 服务路由

所有 AI 服务流量通过 **家庭宽带** 代理组（链式代理）：

- OpenAI (ChatGPT)
- Claude (Anthropic)
- Perplexity AI
- Coze

配置示例：
```yaml
rules:
  - DOMAIN-SUFFIX,claudeusercontent.com,AIService
  - DOMAIN-SUFFIX,coze.com,节点选择
  - DOMAIN-SUFFIX,perplexity.ai,AIService
  - RULE-SET,OpenAI,AIService
  - RULE-SET,Claude,AIService
  - RULE-SET,AI,AIService
```

## 常见问题

### Q: 链式代理是什么？
A: 链式代理（Dialer Proxy）是指流量先经过一个代理，再通过第二个代理出站。本配置中，ISP 家宽节点会先连接到"服务器"组，再进行最终连接。

### Q: 为什么不能使用 relay 策略组？
A: **Mihomo 内核已在 v1.19.17（2024年）完全移除了 relay 策略组**。原因包括：
- WireGuard 协议不支持 relay 模式
- dialer-proxy 提供了更灵活的底层实现
- 简化代码库，减少维护成本

如果你的配置文件使用了 relay，需要迁移到 dialer-proxy，否则会导致配置加载失败。

### Q: 如何从 Relay 迁移到 Dialer-Proxy？
A:

**旧配置（不可用）**：
```yaml
proxy-groups:
  - name: 代理链
    type: relay
    proxies:
      - 前置代理
      - 出口代理
```

**新配置（推荐）**：
```yaml
proxy-providers:
  出口节点:
    type: http
    url: https://example.com/sub
    override:
      dialer-proxy: "前置代理"
```

### Q: 为什么 AI 服务使用家庭宽带？
A: 家庭宽带（ISP）提供住宅 IP，相比数据中心 IP 更不容易被 AI 服务识别和限制。

### Q: 如何启用 UDP？
A: 删除或注释掉配置文件末尾的这两条规则：
```yaml
# - AND,((DST-PORT,443),(NETWORK,UDP)),REJECT
# - NETWORK,UDP,REJECT
```

### Q: 如何添加自定义规则？
A: 在 `rules:` 部分添加规则，格式参考现有规则。建议在最终规则前添加。

### Q: 链式代理会影响速度吗？
A: 会增加一定延迟，因为流量需要经过两层代理。建议仅对特定服务（如 AI 服务）使用链式代理。

### Q: 我的客户端提示 "unknown proxy group type: relay"，怎么办？
A: 这说明你的 Mihomo 内核版本 ≥ v1.19.17，已不支持 relay。请使用 **ClashConf-DialerProxy.yaml** 配置文件。

## 文件结构

```
.
├── ClashConf-DialerProxy.yaml      # ✅ 主配置文件（dialer-proxy 实现）
├── ClashConf-Relay.yaml            # ⚠️ 旧版配置（relay 实现，已过时）
├── profiles/                       # 自动生成的缓存目录
│   ├── ProxyProvider/              # 代理提供者缓存
│   │   ├── USA-ISP-Collection.yaml
│   │   ├── ServerGroup.yaml
│   │   └── ServerBuckup.yaml
│   └── RuleProvider/               # 规则提供者缓存
│       └── Media/                  # 媒体规则
└── README.md                       # 说明文档
```

### 配置文件选择指南

| Mihomo 内核版本 | 推荐配置文件 | 说明 |
|----------------|-------------|------|
| v1.19.17+ | ✅ ClashConf-DialerProxy.yaml | relay 已移除，必须使用 dialer-proxy |
| v1.18.6 - v1.19.16 | ✅ ClashConf-DialerProxy.yaml | relay 已弃用，建议使用 dialer-proxy |
| v1.18.5 及以下 | ⚠️ ClashConf-Relay.yaml | 旧版内核可用，但建议升级 |

## 更新日志

查看 Git 提交历史：

### 2025-02
- **重构配置文件**：从 relay 策略组迁移到 dialer-proxy 实现
- **适配 Mihomo v1.19.17+**：移除已废弃的 relay 支持
- **完善文档**：添加 relay 移除背景说明和迁移指南
- 禁用 UDP 连接

### 历史更新
- 支持链式代理配置
- 新增 AI 服务专用路由
- 优化 DNS 配置

## 注意事项

⚠️ **重要提醒**：

1. **内核版本要求**：
   - 使用 **ClashConf-DialerProxy.yaml** 需要 Mihomo v1.18.6+ 内核
   - Mihomo v1.19.17+ 已完全移除 relay，无法使用 ClashConf-Relay.yaml
   - 推荐使用最新版 Mihomo 内核

2. **替换订阅链接**：必须将示例链接替换为实际订阅地址

3. **修改 API 密码**：避免使用默认密码 `password`

4. **链式代理性能**：使用 dialer-proxy 会增加延迟，建议仅对特定服务使用

5. **UDP 禁用影响**：某些应用可能需要 UDP（如游戏、语音通话），请根据需求调整

6. **规则更新**：配置会自动更新在线规则集，每 24 小时一次

## 兼容性说明

### 支持的客户端

以下客户端使用 Mihomo 内核，支持 **ClashConf-DialerProxy.yaml**：

| 客户端 | 平台 | 支持 dialer-proxy | 推荐 |
|--------|------|------------------|------|
| [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev) | Windows/macOS/Linux | ✅ | ⭐⭐⭐⭐⭐ |
| [Mihomo Party](https://github.com/mihomo-party-org/mihomo-party) | Windows/macOS/Linux | ✅ | ⭐⭐⭐⭐⭐ |
| [FlClash](https://github.com/chen08209/FlClash) | Android/Windows/macOS/Linux | ✅ | ⭐⭐⭐⭐ |
| [ClashX Meta](https://github.com/MetaCubeX/ClashX.Meta) | macOS | ✅ | ⭐⭐⭐⭐ |

### 不兼容的客户端

以下客户端使用旧版 Clash Premium 内核，**不支持** dialer-proxy：

- ❌ Clash for Windows (CFW) - 已停止维护
- ❌ ClashX (原版) - 已停止维护
- ❌ Clash for Android (CFA) - 已停止维护

如果你仍在使用这些客户端，建议迁移到上述 Mihomo 内核的客户端。

## 许可证

本配置文件基于社区规则集构建，规则来源：
- [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)
- [dler-io/Rules](https://github.com/dler-io/Rules)
- [blackmatrix7](https://github.com/blackmatrix7/ios_rule_script)

## 贡献

欢迎提交 Issue 和 Pull Request 来改进此配置。

---

**⚡ 提示**：首次使用前，请务必修改订阅链接和 API 密码！

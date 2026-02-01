# Ar-ClashConf

Clash 代理配置文件集合，支持链式代理（Dialer Proxy）功能，适用于需要多层代理的场景。

## 配置文件说明

### ClashConf-DialerProxy.yaml

完整的 Clash 配置文件，包含链式代理支持和丰富的分流规则。

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

配置文件中 `ISP-USA-Collection` 使用了链式代理：

```yaml
ISP-USA-Collection:
  override:
    dialer-proxy: "服务器"  # 通过"服务器"组进行二次代理
```

**工作原理**：流量路径为 `本地 → 服务器组 → ISP 家宽节点 → 目标网站`

**适用场景**：
- 使用住宅 IP 访问对数据中心 IP 敏感的服务
- 需要隐藏真实出口节点的场景
- AI 服务访问（降低被封禁风险）

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

## 文件结构

```
.
├── ClashConf-DialerProxy.yaml      # 主配置文件（链式代理版本）
├── profiles/                       # 自动生成的缓存目录
│   ├── ProxyProvider/              # 代理提供者缓存
│   │   ├── USA-ISP-Collection.yaml
│   │   ├── ServerGroup.yaml
│   │   └── ServerBuckup.yaml
│   └── RuleProvider/               # 规则提供者缓存
│       └── Media/                  # 媒体规则
└── README.md                       # 说明文档
```

## 更新日志

查看 Git 提交历史：

- **最新提交**：禁用 UDP 连接
- 支持链式代理配置
- 新增 AI 服务专用路由
- 优化 DNS 配置

## 注意事项

⚠️ **重要提醒**：

1. **替换订阅链接**：必须将示例链接替换为实际订阅地址
2. **修改 API 密码**：避免使用默认密码 `password`
3. **链式代理性能**：使用 dialer-proxy 会增加延迟，建议仅对特定服务使用
4. **UDP 禁用影响**：某些应用可能需要 UDP（如游戏、语音通话），请根据需求调整
5. **规则更新**：配置会自动更新在线规则集，每 24 小时一次

## 许可证

本配置文件基于社区规则集构建，规则来源：
- [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)
- [dler-io/Rules](https://github.com/dler-io/Rules)
- [blackmatrix7](https://github.com/blackmatrix7/ios_rule_script)

## 贡献

欢迎提交 Issue 和 Pull Request 来改进此配置。

---

**⚡ 提示**：首次使用前，请务必修改订阅链接和 API 密码！

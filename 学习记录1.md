# Clash Verge Windows 学习手册

本文面向 Windows 下使用 `Clash Verge Rev` 的入门与进阶场景，结合你当前的配置目录
`C:\Users\wh\AppData\Roaming\io.github.clash-verge-rev.clash-verge-rev`
进行说明。

## 0. 先给一个总览

Clash Verge Rev 可以理解成两层：

- 上层是桌面应用：负责界面、订阅管理、系统代理、策略组切换、日志查看。
- 下层是 Mihomo/Clash 内核：负责真正的分流、DNS 处理、节点连接、转发流量。

你当前配置里已经能看出几件事：

- 本地混合代理端口是 `7897`
- SOCKS 端口是 `7898`
- HTTP 端口是 `7899`
- 当前 `mode: rule`
- 已启用系统代理
- 已启用 DNS 增强配置
- 当前未开启 `TUN`

## 1. 配置目录详细介绍，以及如何正确配置

### 1.1 目录用途总览

你当前的配置目录大致可以分为 5 类：

- 应用设置
- 订阅索引
- 远程订阅原始文件
- 本地增强文件
- 运行期生成文件与日志

### 1.2 关键文件说明

#### `config.yaml`

这是内核基础配置，决定本地监听端口、模式、控制器、TUN 等。

你当前这份配置的重点：

```yaml
mixed-port: 7897
socks-port: 7898
port: 7899
mode: rule
tun:
  enable: false
```

含义：

- `mixed-port: 7897`：混合代理端口，通常同时兼容 HTTP 和 SOCKS 客户端，是最常用的入口。
- `socks-port: 7898`：纯 SOCKS 入口。
- `port: 7899`：HTTP 代理端口。
- `mode: rule`：按规则分流。
- `tun.enable: false`：当前没有启用 TUN 接管。

#### `verge.yaml`

这是 Clash Verge Rev 应用自身的行为配置，不是订阅内容本身。

你当前这份里比较重要的项：

```yaml
enable_system_proxy: true
enable_dns_settings: true
clash_core: verge-mihomo
auto_check_update: true
```

含义：

- `enable_system_proxy: true`：启用系统代理。浏览器、部分 GUI 程序、部分 CLI 会走系统代理。
- `enable_dns_settings: true`：启用单独的 DNS 配置。
- `clash_core: verge-mihomo`：使用的内核是 Mihomo 变体。
- `auto_check_update: true`：这是应用检查更新，不是订阅自动更新。

#### `profiles.yaml`

这是最重要的“索引文件”，负责描述当前启用了哪个订阅、订阅关联了哪些增强文件、是否允许自动更新等。

你的目录里可以看到一条远程订阅：

```yaml
current: R2Gi4KwsGNL1
...
- uid: R2Gi4KwsGNL1
  type: remote
  name: 魔戒.net
  file: R2Gi4KwsGNL1.yaml
```

同时，它还把多个增强文件挂到了这条订阅上：

```yaml
option:
  merge: m8wOvGqhh5I3
  script: sxQP9zjANZTE
  rules: rkcF5ucDFcNF
  proxies: pCYIZHNdJai7
  groups: gItEeBwCBbbp
```

这表示：

- 主订阅文件是 `profiles\R2Gi4KwsGNL1.yaml`
- 自定义规则在 `profiles\rkcF5ucDFcNF.yaml`
- 自定义代理组在 `profiles\gItEeBwCBbbp.yaml`
- 自定义节点在 `profiles\pCYIZHNdJai7.yaml`
- 额外合并配置在 `profiles\m8wOvGqhh5I3.yaml`
- 脚本二次处理在 `profiles\sxQP9zjANZTE.js`

#### `dns_config.yaml`

这是 DNS 增强配置。

你当前的重点是：

```yaml
enable: true
listen: :53
enhanced-mode: fake-ip
```

含义：

- 开启了 Clash DNS
- 本地监听 53 端口
- 使用 `fake-ip` 方案

`fake-ip` 的好处是对分流友好，尤其适合浏览器和复杂规则场景；但它也意味着 DNS 流量和应用直觉看到的 IP 不一定是一回事，实际是由 Clash 在内部做域名映射和转发。

#### `profiles\R2Gi4KwsGNL1.yaml`

这是远程订阅拉下来的主配置。通常很大，里面包含：

- `proxies`
- `proxy-groups`
- `rules`
- 有时还会带 `dns`、`rule-providers`

不建议你直接改这个文件，因为：

- 一刷新订阅就可能被覆盖
- 一旦订阅更新，手改内容容易丢

#### `profiles\gItEeBwCBbbp.yaml`

这是自定义代理组增强文件。

适合放：

- 新增策略组
- 调整策略组显示顺序
- 删除订阅自带的某些组

#### `profiles\rkcF5ucDFcNF.yaml`

这是自定义规则增强文件。

适合放：

- 某个服务的域名分流
- 某个进程的分流
- 自定义直连或代理规则

#### `profiles\pCYIZHNdJai7.yaml`

这是自定义节点增强文件。

适合放：

- 手动新增单独节点
- 删除订阅中的某些节点
- 对个别节点做补充

#### `clash-verge.yaml` / `clash-verge-check.yaml`

这是生成后的最终配置文件。通常是应用把基础配置、订阅配置、增强文件合并后的结果。

不要优先手改这两个文件，因为它们通常会被应用重新生成。

#### `logs\`

这是日志目录。

- 顶层 `logs\*.log`：应用日志
- `logs\service\*.log`：服务或内核日志

遇到“分流不对”“节点连不上”“CLI 登录失败”等问题时，日志非常有用。

### 1.3 正确配置的原则

推荐遵循下面这套原则：

1. 不要直接改远程订阅主文件 `profiles\R2Gi4KwsGNL1.yaml`
2. 优先改增强文件：`groups`、`rules`、`proxies`、`merge`
3. 改完后重载配置，确认生成文件已更新
4. 规则里引用的代理组名称必须和组名完全一致
5. 不要把 `profiles.yaml`、生成配置、日志随便发给别人，里面可能包含订阅地址、流量信息、节点参数

### 1.4 常见配置任务应该改哪个文件

| 需求 | 推荐修改文件 |
| --- | --- |
| 调整本地监听端口、模式、TUN | `config.yaml` |
| 调整应用行为、系统代理、界面选项 | `verge.yaml` |
| 禁止订阅自动更新 | `profiles.yaml` |
| 新增某个服务的专属策略组 | `profiles\gItEeBwCBbbp.yaml` |
| 给某个服务加分流规则 | `profiles\rkcF5ucDFcNF.yaml` |
| 手动新增节点 | `profiles\pCYIZHNdJai7.yaml` |
| 最终运行效果检查 | `clash-verge.yaml` |

## 2. Windows 下流量如何进出，以及为什么你会遇到“浏览器能授权、CLI 登录失败”

### 2.1 最核心的流量路径

在 Windows 下，最常见的流量路径如下：

#### 情况 A：应用遵守系统代理

```text
应用程序 -> Windows 系统代理 -> 127.0.0.1:7897/7899/7898 -> Clash/Mihomo -> 规则判断 -> 选中节点 -> 目标网站
```

这类程序通常包括：

- 大多数浏览器
- 很多 Electron 应用
- 一部分 CLI 工具

#### 情况 B：应用不遵守系统代理

```text
应用程序 -> 直接联网
```

这种情况下，即使 Clash 已经打开、系统代理也启用了，程序仍可能直连。  
要让这类程序走代理，通常需要以下方式之一：

- 手动设置 `HTTP_PROXY` / `HTTPS_PROXY` / `ALL_PROXY`
- 应用自身提供代理设置项
- 开启 `TUN`，让系统层面接管更多流量

### 2.2 你当前这套配置的具体含义

你当前的关键点是：

- `mode: rule`
- `enable_system_proxy: true`
- `tun.enable: false`

这意味着你当前主要依赖的是：

1. 程序先愿意把流量发给本地代理
2. 然后 Clash 再按照规则决定走哪个策略组

也就是说，**你现在并不是“系统所有流量都被强制接管”**，而是“能走进 Clash 的流量，再由 Clash 继续分流”。

### 2.3 规则模式、全局模式、直连模式的差别

#### `rule`

按规则分流：

- 命中某条规则，就走对应策略组
- 没命中时，就走规则末尾的默认动作

这是平时最推荐的模式，但前提是规则要写对。

#### `global`

所有进入 Clash 的流量都走同一个全局代理出口。

好处：

- 不容易因为漏规则导致某些接口直连

坏处：

- 非目标服务的流量也会一起走代理
- 很多本地或国内服务不一定适合都代理

#### `direct`

所有进入 Clash 的流量都不代理，直接出去。

### 2.4 为什么你会出现“浏览器授权成功，但 CLI 登录失败；切到全局模式后成功”

根据你描述的现象，比较大的概率是下面这个原因：

#### 更可能的真实原因

CLI 的流量其实已经进入 Clash 了，但在 `rule` 模式下，**Claude/Anthropic 相关的某些域名没有被正确分流到代理组**，导致：

- 浏览器授权页面能打开
- 但 CLI 后续调用的某些接口走成了直连，或走到了错误的组
- 最终登录流程不完整，于是失败

你切成 `global` 后，所有进入 Clash 的流量都统一走代理，所以登录成功。

这也解释了为什么：

- 不是单纯“浏览器能走、CLI 完全不走”
- 而是“CLI 在规则模式下表现不稳定，全局模式下恢复”

#### 为什么不太像“CLI 完全没走代理”

如果 CLI 完全绕过了 Clash，那么你只切换 `rule` / `global`，通常不会让结果发生这么明显的变化。

### 2.5 这类问题的正确处理思路

如果你想让某个服务在 Windows 下稳定工作，建议优先按下面顺序处理：

1. 给该服务单独建立策略组
2. 把该服务的域名明确分流到这个策略组
3. 这个策略组内尽量只放同一地区的节点
4. 实际使用时固定到一个稳定节点，不要频繁切换
5. 如果某些 CLI 仍不稳定，再考虑环境变量代理或 TUN

你当前给 Claude 新增单独策略组，本质上就是在做这件事。

## 3. 如何确认当前是否真的在使用目标地区、目标节点

这个问题要分成两个层次确认：

- 逻辑层：规则是否命中了你想要的策略组
- 出口层：最终出去的公网 IP 是否真的是目标地区

### 3.1 在 Clash Verge Rev 面板里确认

至少确认这几件事：

1. 当前运行模式是不是你期望的模式，例如 `rule`
2. 目标策略组当前实际选中了哪个节点
3. 连接列表里目标域名是否命中了你期望的策略组

例如，对 Claude 来说，你应该确认：

- `Claude` 组当前选中的就是某个美国节点
- `claude.ai`、`api.anthropic.com` 等连接命中了 `Claude`

### 3.2 用外部 IP 查询验证出口地区

仅看策略组名字不够，因为“美国节点”这个名字并不能 100% 等于真实美国出口。

你应该做实际出口验证。

#### 浏览器验证

打开下面任意一个网站：

- `https://ipinfo.io`
- `https://ifconfig.me`
- `https://ip.sb`

看：

- IP 地址
- 国家/地区
- ASN/运营商

#### PowerShell 验证

如果程序走的是系统代理或你手动设置了代理环境变量，可以用：

```powershell
curl https://ipinfo.io/json
```

如果你想显式指定本地 SOCKS 代理测试：

```powershell
curl.exe --proxy socks5h://127.0.0.1:7898 https://ipinfo.io/json
```

如果你想显式指定 HTTP 代理测试：

```powershell
curl.exe --proxy http://127.0.0.1:7899 https://ipinfo.io/json
```

看返回里的国家、城市、IP 是否符合你的预期。

### 3.3 用连接日志确认“是哪一个节点在处理”

最稳妥的方法是同时看两样：

1. 连接日志里，域名是否进入了目标策略组
2. 目标策略组当前选中的是哪个具体节点

如果这两者都正确，说明逻辑上已经命中你想要的出口。

### 3.4 如何减少“明明选了美国节点，但看起来不像美国”的情况

建议：

- 尽量使用单节点手动固定，不要让 `Claude` 这类敏感服务走 `url-test` 自动切换
- 不要在同一会话里频繁切换国家
- 浏览器、CLI、应用内嵌网页尽量统一走同一策略组
- 如果你对某个服务要求高，优先使用长期稳定出口的节点，而不是延迟最低但频繁波动的节点

## 4. 如何新增一个策略组以及对应节点，例如给 Claude 单独配置

### 4.1 基本原理

新增一个服务专属策略，通常分 2 步：

1. 在 `groups` 增强文件里新增策略组
2. 在 `rules` 增强文件里把该服务域名指向这个策略组

如果订阅里没有你想用的节点，再额外改 `proxies` 增强文件。

### 4.2 你当前目录中对应的文件

- 自定义代理组：`profiles\gItEeBwCBbbp.yaml`
- 自定义规则：`profiles\rkcF5ucDFcNF.yaml`
- 自定义节点：`profiles\pCYIZHNdJai7.yaml`

### 4.3 示例：给 Claude 单独新增一个策略组

这一节的重点不是“必须真的再新增一个 Claude 组”，而是用 `Claude` 当示例，说明如何给敏感服务做专属策略，以尽量降低因为出口地区频繁变化、节点频繁切换带来的风控概率。

假设你还没有单独的 Claude 组，或者你想重新整理现有 Claude 配置，可以把组名设为 `✦ Claude`。

先改 `profiles\gItEeBwCBbbp.yaml`：

```yaml
prepend:
  - { name: ✦ Claude, type: select, proxies: [美国USLA-A, 美国LA-优化-GPT, 美国LA-优化2-GPT, 美国LA-优化3-GPT] }

append: []

delete: []
```

然后改 `profiles\rkcF5ucDFcNF.yaml`，在 `prepend` 中增加 Claude 规则：

```yaml
prepend:
  - 'DOMAIN-SUFFIX,claude.ai,✦ Claude'
  - 'DOMAIN-SUFFIX,anthropic.com,✦ Claude'
  - 'DOMAIN,api.anthropic.com,✦ Claude'
  - 'DOMAIN,console.anthropic.com,✦ Claude'
  - 'DOMAIN-SUFFIX,claudeusercontent.com,✦ Claude'

append: []

delete: []
```

注意：

- 规则里的目标组名必须和代理组名完全一致
- 如果你已经有一个叫 `Claude` 的组，而不是 `✦ Claude`，那规则里也必须继续写 `Claude`
- 对 Claude 这类服务，更推荐 `select` + 固定单个美国节点，而不是自动测速切换

### 4.4 如果订阅里没有你想放进组里的节点

那就改 `profiles\pCYIZHNdJai7.yaml`，往里面手动追加节点，再把这些节点名写入 `✦ Claude` 或 `Claude` 组。

常见思路是：

- `groups` 文件负责“组”
- `proxies` 文件负责“节点”
- `rules` 文件负责“什么流量进哪个组”

### 4.5 修改后要做什么

每次改完增强文件后，都要做下面几步：

1. 在 Clash Verge Rev 里重载配置或重新应用订阅
2. 打开连接日志，验证目标域名是否命中预期组
3. 打开 IP 查询网站，验证出口地区是否正确

## 5. 关于你当前 Claude 配置的建议

你当前思路是对的：给 Claude 单独建一个组，并且尽量只使用美国节点。

更推荐的实践是：

1. `Claude` 组用 `select`，不要用自动测速切换
2. 组内最多放几个美国节点作为备选
3. 实际使用时固定到一个长期稳定节点
4. Claude 相关流量都指向这个组
5. 浏览器和 CLI 尽量不要在同一账号会话中频繁切换国家和节点

这能明显降低“不同请求从不同国家或不同 IP 出去”导致的不稳定问题。

需要说明的是，任何代理配置都不能保证第三方平台绝不会风控，但“单独策略组 + 同地区稳定节点 + 规则明确命中”是更稳妥的做法。

## 6. 结合你当前配置的几个结论

基于你当前目录里的配置，可以得到这些结论：

1. 你目前主要依赖的是系统代理，不是 TUN 全量接管
2. 你当前内核模式是 `rule`
3. 你已经开始使用“增强文件”方式管理自定义逻辑，这是正确方向
4. 你后续新增 Claude、ChatGPT 这类专属策略时，应继续优先修改增强文件，而不是手改远程订阅主文件

## 7. 推荐操作清单

如果你后续要长期维护这套配置，推荐按这个顺序操作：

1. 先在面板确认当前模式、当前节点
2. 再改增强文件
3. 改完立即重载
4. 用连接日志确认规则命中
5. 用 IP 查询确认出口地区
6. 重要服务固定在单独策略组，且尽量固定国家

## 8. 速查版

### 禁止订阅自动更新

改 `profiles.yaml`：

```yaml
allow_auto_update: false
```

### 新增策略组

改 `profiles\gItEeBwCBbbp.yaml`

### 新增服务分流规则

改 `profiles\rkcF5ucDFcNF.yaml`

### 新增自定义节点

改 `profiles\pCYIZHNdJai7.yaml`

### 验证最终是否生效

看 Clash Verge Rev 面板、连接日志、IP 查询结果，必要时检查 `clash-verge.yaml`

---

如果你后续想继续扩展，我建议下一步做两件事：

- 把 `Claude` 组名和规则目标名统一成你最终想要的显示样式，例如 `✦ Claude`
- 为 Claude 单独固定一个长期稳定的美国节点，减少同账号多地区切换

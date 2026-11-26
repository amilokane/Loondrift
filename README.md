<p align="center">
  <img src="assets/loondrift-banner.jpg" alt="Loondrift" width="900">
</p>

<p align="center">
  <strong>为 iPhone 准备的 Loon 和 Egern 分流配置</strong><br>
  国内站点通常直连 · 海外 AI 独立选路 · 域名广告拦截
</p>

<p align="center">
  <a href="#egern-配置"><strong>Egern 配置</strong></a>　·　<a href="#loon-配置"><strong>Loon 配置</strong></a>　·　<a href="#ip-归属地规则">IP 归属地规则</a>　·　<a href="#上游来源">上游来源</a>
</p>

## 仓库内容

| 文件 | 适用 App | 说明 |
| :--- | :--- | :--- |
| [`Egern.yaml`](Egern.yaml) | Egern | 主配置，已内置下面的 Egern 模块。 |
| [`Loon.conf`](Loon.conf) | Loon | 主配置。 |
| [`plugin/ip-attribution.yaml`](plugin/ip-attribution.yaml) | Egern | IP 归属地规则模块。 |
| [`plugin/ip-attribution.plugin`](plugin/ip-attribution.plugin) | Loon | IP 归属地规则插件，需要单独导入。 |

两份主配置各自维护，版本号写在各自文件的首行。

## Egern 配置

### 导入

在 Egern 首页选择从 URL 导入，粘贴下面的地址。

```text
https://raw.githubusercontent.com/amilokane/Loondrift/main/Egern.yaml
```

1. 添加自己的 Egern 兼容订阅，它会归入隐藏的 `Airport` 组。订阅里的流量、到期、官网等提示条目会被自动排除。
2. 在 `Manual` 里查看和选择全部节点。`Proxy` 初始使用 `Smart`，多数服务组初始跟随 `Proxy`。
3. 要让 App 去广告模块生效，在 iOS 安装并信任自己的 Egern 证书。

> [!NOTE]
> `Smart` 使用 Egern 的 `smart` 策略组类型。导入报错或 `Smart` 里没有节点时，先把 Egern 升级到支持该类型的版本。

### 策略组

| 策略组 | 初始选择 | 说明 |
| :--- | :--- | :--- |
| **Proxy** | `Smart` | 主出口。Google、YouTube、GitHub、Telegram、Meta、Twitter、Netflix、Spotify 初始跟随它。 |
| **AI** | `Proxy` | 聚合海外 AI 规则（涵盖 ChatGPT、Claude、Google AI、Perplexity、Grok、Cursor、Copilot 等）。DeepSeek、豆包等国内 AI 按规则直连。 |
| **TikTok** | `Japan` | TikTok 在香港节点不可用，初始避开香港。 |
| **Apple · Microsoft · PayPal** | `DIRECT` | 需要走节点时在组内切换。 |
| **IP Location** | `Proxy` | 内置的 IP 归属地模块使用这个出口。 |
| **AdBlock** | `REJECT` | 控制两份广告域名规则，切到 `DIRECT` 即放行。 |
| **Final** | `Proxy` | 处理没有命中任何规则的请求。 |

各组的可选项顺序相同，依次是 `Smart`、香港、日本、美国、新加坡、台湾、`Other Regions`、`Manual`。规则集没有收录、但 IP 位于中国大陆的目标会直连。

> [!TIP]
> ChatGPT 和 Claude 在香港节点不可用。`Smart` 选中香港节点时，把 `AI` 切到其他地区或 `Manual`。

<details>
<summary><strong>地区组与节点识别</strong></summary>

- 香港、日本、美国、新加坡、台湾五个地区组按节点名称筛选，各自选用延迟最低的节点。
- 地区组每 600 秒向 `https://cp.cloudflare.com/generate_204` 测速一次，单次超时 5 秒，延迟差超过 50 毫秒才切换。
- 名称识别支持旗帜、中英文地名、机场三字码（如 LAX、HND、SIN、TPE）和中转线路缩写（如 沪港、深日、广美）。
- `Other Regions` 列出没有命中五个地区的节点，由用户手动选择。

</details>

<details>
<summary><strong>Smart 如何选节点</strong></summary>

`Smart` 综合延迟、抖动和成功率给节点打分，再按名称里的倍率加权。系数越小越优先。

| 节点名称 | 系数 | 效果 |
| :--- | :---: | :--- |
| 标有低于 1 倍（含 0 倍），例如 `0.5x`、`x0.8`、`0.2倍`、`0x` | 0.1 | 优先使用 |
| 标有 1 倍或没有标倍率，例如 `1x`、`1.0倍` | 1 | 正常参与 |
| 标有高于 1 倍或「高倍」，例如 `2x`、`倍率:1.5` | 10 | 尽量不用 |

- 高倍率（`>1`）与低倍率（`<1`）正则互斥；单独标出的 `0.5` 按低倍率处理，`v0.8` 等版本号忽略。
- 名称带「直连」或 `DIRECT` 的节点不参与自动选择，可以在 `Manual` 或地区组里手动选。

[Egern 策略组文档](https://egernapp.com/zh-CN/docs/configuration/policy_groups)

</details>

<details>
<summary><strong>DNS 解析方式</strong></summary>

Egern 接管系统的 DNS 查询，向应用返回真实 IP，不同域名交给对应的加密上游：

| 域名 | 上游 | 连接方式 |
| :--- | :--- | :--- |
| 国内域名、国内 AI、`.cn`、Apple、Microsoft、PayPal | 阿里和腾讯 DoH（`223.5.5.5`、`1.12.12.12`） | 直连 |
| 海外服务与其他未知域名 | Cloudflare 和 Google DoH（`1.1.1.1`、`8.8.8.8`） | 经 `Proxy` |
| 本地域名（`.local`、`.lan`、`.localdomain`、`.home.arpa`） | 系统 DNS | 当前网络 |

- YouTube、TikTok、海外 AI、Google、GitHub、Telegram、Twitter、Meta（Facebook/Instagram/WhatsApp）、Netflix、Spotify 的规则集排在国内集合之前，这些服务的域名始终走海外 DoH。
- 建立代理连接时，Egern 仅通过 `proxy_nameservers` 里的国内加密 DoH（`223.5.5.5`、`1.12.12.12`）直连解析代理节点自身域名。
- 上游全部使用 IP 地址，不需要明文查询上游主机名。

[Egern DNS 文档](https://egernapp.com/zh-CN/docs/configuration/dns)

</details>

### 模块

配置内置 13 个模块，默认全部启用，更新周期统一为 24 小时。

| 类别 | 模块 |
| :--- | :--- |
| App 去广告与增强 | 去广告合集、微博、哔哩哔哩、百度贴吧、斗鱼直播、闲鱼、YouTube 增强、淘宝、拼多多、网易云音乐、懂球帝 |
| 工具 | iRingo WeatherKit、IP Attribution |

- 需要 HTTPS 重写的模块，解密范围使用模块自己声明的域名。

[Egern 模块文档](https://egernapp.com/zh-CN/docs/configuration/modules)

### 版本

版本号写在 [`Egern.yaml` 首行](Egern.yaml#L1)。每次更新小数位加 1，满 10 进位（2.9 之后是 3.0）。

## Loon 配置

### 导入

在 iPhone 的 Loon 中打开「配置」，选择从 URL 导入，粘贴下面的地址。

```text
https://raw.githubusercontent.com/amilokane/Loondrift/main/Loon.conf
```

1. 添加自己的节点订阅，在「默认代理」中选一个可用节点或策略组。
2. 切换到规则模式。想让 AI、YouTube 等服务用不同线路，可以在各自的策略组里选。

> [!TIP]
> **从旧配置更新**　关闭「全覆盖更新」以保留手机上的节点、订阅和证书。更新后刷新远程规则和插件，再确认常用服务的策略选择。

### 策略组

| 策略组 | 初始行为 |
| :--- | :--- |
| **默认代理** | Google、YouTube、Telegram 等服务初始跟随此组。 |
| **AI** | ChatGPT、Claude、Gemini 等海外 AI 初始走「美国」。DeepSeek、豆包等国内 AI 按规则直连。 |
| **Apple · Microsoft** | 初始直连，可在各自的策略组切换。 |
| **漏网之鱼** | 处理未命中规则的请求，初始走「默认代理」。 |

<details>
<summary><strong>选地区或固定节点</strong></summary>

- **按地区选**　香港、日本、美国、新加坡和台湾根据节点名称筛选，并在组内故障转移。德国、英国、韩国等线路可在「其他节点」查找。
- **按测速选**　「自动优选」按测速选择线路，并过滤已识别的高倍率节点。
- **按倍率选**　「低倍率节点」和「高倍率节点」按节点名称里的倍率标记分组，组内手动选择。
- **固定单个节点**　先在服务组里选「手动选择」，再进入该组指定节点。引用「手动选择」的服务会共用当前节点。

</details>

<details>
<summary><strong>启用 App 内去广告</strong></summary>

域名广告拦截由「AdBlock」控制，初始为拦截。B 站、YouTube 等需要解密 HTTPS 的插件，还需要在 iOS 安装并信任 Loon 证书。MitM over HTTP/2 和 QUIC 回退保护可按插件说明设置。

</details>

<details>
<summary><strong>排查通话或联机问题</strong></summary>

配置默认禁用 STUN。如果语音、视频通话或联机遇到问题，可以将 `disable-stun` 改为 `false` 再试。

</details>

### 版本

版本号写在 [`Loon.conf` 首行](Loon.conf#L1)。

## IP 归属地规则

把微博、哔哩哔哩、知乎、抖音、小红书等三十多个 App 里与 IP 属地相关的请求交给指定策略。规则共 242 条，其中 193 条走指定策略，28 条直连，21 条拒绝。规则转换自 [SunsetMkt/anti-ip-attribution](https://github.com/SunsetMkt/anti-ip-attribution)，并加入了本地补充。

| App | 文件 | 使用方式 |
| :--- | :--- | :--- |
| Egern | [`plugin/ip-attribution.yaml`](plugin/ip-attribution.yaml) | `Egern.yaml` 已内置。出口由 `IP Location` 策略组决定，初始跟随 `Proxy`。 |
| Loon | [`plugin/ip-attribution.plugin`](plugin/ip-attribution.plugin) | `Loon.conf` 没有内置，需要单独导入。导入后把 `PROXY` 绑定到想用的策略。 |

Loon 插件地址：

```text
https://raw.githubusercontent.com/amilokane/Loondrift/main/plugin/ip-attribution.plugin
```

在其他 Egern 配置里单独使用模块时，导入下面的地址。模块参数 `PROXY` 是这些规则使用的策略名称，默认值为 `Proxy`。

```text
https://raw.githubusercontent.com/amilokane/Loondrift/main/plugin/ip-attribution.yaml
```

## 上游来源

| 内容 | 来源 |
| :--- | :--- |
| Egern 服务与广告规则 | [Repcz/EgernRules](https://github.com/Repcz/EgernRules)，由 blackmatrix7 的规则转换为原生 YAML |
| Loon 服务与广告规则 | [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Loon) |
| 国内与海外 AI 规则 | [VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules) |
| Threads 列表 | [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat/tree/meta/geo/geosite/classical) |
| Egern 广告域名补充 | [RiverFlowsInUUU/RFU-Rules](https://github.com/RiverFlowsInUUU/RFU-Rules) |
| Egern 模块 | [Egern Hub 模块目录](https://egern.pages.dev/)、[IBL3ND/module](https://github.com/IBL3ND/module) |
| Loon 插件 | [可莉插件中心](https://hub.kelee.one/) |
| IP 归属地规则 | [SunsetMkt/anti-ip-attribution](https://github.com/SunsetMkt/anti-ip-attribution) |
| 策略组图标 | [AIsouler/MyClash](https://github.com/AIsouler/MyClash) |

`plugin/` 目录下的规则沿用上游的 GPL-3.0 许可，完整文本见 [`plugin/LICENSE`](plugin/LICENSE)。

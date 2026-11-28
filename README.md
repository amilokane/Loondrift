<div align="center">

<img src="assets/loondrift-banner.jpg" alt="Loondrift" width="840" />

<h1>Loondrift</h1>

<p><strong>适用于 Egern 与 Loon 的轻量 iOS 分流配置</strong></p>

<p>
  <a href="Egern.yaml"><img src="https://img.shields.io/badge/Egern-v3.2-0d9488?style=flat-square&logo=apple&logoColor=white" alt="Egern v3.2" /></a>
  <a href="Loon.conf"><img src="https://img.shields.io/badge/Loon-v1.0.3-0284c7?style=flat-square&logo=safari&logoColor=white" alt="Loon v1.0.3" /></a>
  <a href="plugin/ip-attribution.yaml"><img src="https://img.shields.io/badge/IP%20Attribution-242%20Rules-10b981?style=flat-square" alt="IP Attribution" /></a>
  <a href="plugin/LICENSE"><img src="https://img.shields.io/badge/Plugin-GPL--3.0-334155?style=flat-square" alt="GPL-3.0" /></a>
</p>

<p>
  <a href="#快速导入">快速导入</a> &nbsp;/&nbsp;
  <a href="#egern-配置">Egern 配置</a> &nbsp;/&nbsp;
  <a href="#loon-配置">Loon 配置</a> &nbsp;/&nbsp;
  <a href="#ip-归属地分流">IP 归属地</a> &nbsp;/&nbsp;
  <a href="#上游致谢">上游致谢</a>
</p>

</div>

---

## 核心特性

| 特性 | 说明 |
| :--- | :--- |
| **Egern Fake IP 与加密 DNS** | 普通域名使用 Fake IP，局域网后缀返回真实 IP。常规解析按域名分流到国内直连 DoH 或经代理的海外 DoH。 |
| **智能倍率权重与自动测速** | `Smart` 按延迟、抖动与成功率评分并加权 `<1x` / `1x` / `>1x` 倍率；五大主力地区每 10 分钟自动测速选优 |
| **AI 与主流服务精细分流** | DeepSeek、豆包等国内 AI 自动直连；ChatGPT、Claude、Gemini、Grok、Cursor、Copilot 等海外 AI 独立聚合 |
| **开箱即用去广告与增强** | Egern 内置 13 个精选模块（App 去广告、YouTube 增强、WeatherKit、IP 归属地），统一 24h 自动更新 |

---

## 快速导入

复制对应 Raw 链接在客户端内选择 **从 URL 导入** 即可（两份配置独立维护，版本号见各文件首行）：

| 客户端 | 文件 | 版本 | 远程导入链接 |
| :--- | :--- | :---: | :--- |
| **Egern** | [`Egern.yaml`](Egern.yaml) | `3.2` | `https://raw.githubusercontent.com/amilokane/Loondrift/main/Egern.yaml` |
| **Loon** | [`Loon.conf`](Loon.conf) | `1.0.3` | `https://raw.githubusercontent.com/amilokane/Loondrift/main/Loon.conf` |
| **Loon 插件** | [`plugin/ip-attribution.plugin`](plugin/ip-attribution.plugin) | — | `https://raw.githubusercontent.com/amilokane/Loondrift/main/plugin/ip-attribution.plugin` |

> [!NOTE]
> `plugin/ip-attribution.yaml` 已内置于 `Egern.yaml`，Egern 用户无需重复导入。

---

## Egern 配置

### 上手步骤

1. **导入配置**：Egern 首页选择 **从 URL 导入**，粘贴上方 `Egern.yaml` 地址。
2. **添加订阅**：导入自己的订阅，自动归入隐藏的 `Airport` 组并过滤流量、到期、官网等提示条目。
3. **开启 MitM**：在 Egern 生成、安装并在 iOS 系统中信任证书，开启全局 MitM 以启用去广告与增强模块。

> [!TIP]
> - `Smart` 策略组需使用支持 `smart` 类型的最新版 Egern；若无节点请先升级客户端。
> - ChatGPT、Claude、Gemini 等海外 AI 常屏蔽香港节点；当 `Smart` 选到香港时，直接在 `AI` 组切至 `Manual` 或美 / 日 / 新 / 台即可。

### 策略组一览（25 个可见组）

按日常操作频率自上而下排布，高频总控置顶：

| 层级 | 策略组 | 默认出口 | 说明 |
| :--- | :--- | :---: | :--- |
| **总控** | `Proxy` / `Manual` | `Smart` / 全节点 | 主出口与全量手动选节点，首页置顶 |
| **高频** | `AI` / `YouTube` / `TikTok` / `Telegram` / `Discord` | `Proxy`（TikTok 默认 `Japan`） | 海外 AI 聚合与高频切区服务 |
| **海外** | `Twitter` / `Meta` / `Netflix` / `Spotify` / `Google` / `GitHub` | `Proxy` | `Meta` 覆盖 Threads、Facebook、Instagram、WhatsApp |
| **直连** | `Apple` / `Microsoft` / `PayPal` | `DIRECT` | 默认直连降延迟与风控，可随时切代理 |
| **兜底** | `AdBlock` / `Final` | `REJECT` / `Proxy` | 广告总开关与未命中兜底 |
| **节点池** | `Smart` / `Hong Kong` / `Japan` / `United States` / `Singapore` / `Taiwan` / `Other Regions` | 自动优选 / 手动 | 五大地区 600s 自动测速（5s 超时 / 50ms 容差），其余归入 `Other Regions` |

<details>
<summary><strong>Smart 倍率权重与地区识别规则</strong></summary>

<br />

`Smart` 将节点综合评分乘以倍率系数（**系数越小越优先**），自动排除名称含「直连」或 `DIRECT` 的节点：

| 节点倍率特征 | 系数 | 调度策略 |
| :--- | :---: | :--- |
| `<1x`（含 `0x`），如 `0.5x`、`x0.8`、`0.2倍`、`0x`、`0.5` | `0.1` | 优先选用 |
| `1x` 或未标倍率（自动忽略 `v0.8` 等版本号） | `1` | 正常参与 |
| `>1x` 或含「高倍」，如 `1.5x`、`2x`、`倍率:3` | `10` | 尽量避开 |

五大地区自动识别国旗、繁简中英文地名、机场三字码（如 `LAX`、`HND`、`SIN`、`TPE`）及中转简称（如 `沪港`、`深日`、`广美`）。

</details>

<details>
<summary><strong>Fake IP、局域网例外与 DNS 分流</strong></summary>

<br />

启用 `hijack_dns: ['*']`。普通域名使用 Fake IP，以下四个局域网后缀及其子域名使用真实 IP。所有 DoH 上游均直接使用 IP 地址，连接这些上游无需明文引导查询。

| 匹配范围 | 上游 DNS | 路径 |
| :--- | :--- | :---: |
| `.local` / `.lan` / `.localdomain` / `.home.arpa` | 系统 DNS | 当前网络 |
| 国内 AI / `ChinaMax` / `.cn` / Apple / Microsoft / PayPal | 阿里 & 腾讯 DoH（`223.5.5.5` / `1.12.12.12`） | `DIRECT` |
| 海外服务规则集 & 其余未知域名 | Cloudflare & Google DoH（`1.1.1.1` / `8.8.8.8`） | 经 `Proxy` |
| Egern 的代理 DNS 查询（`proxy_nameservers`，跳过 `forward`） | 阿里 & 腾讯 DoH（`223.5.5.5` / `1.12.12.12`） | 强制直连 |

普通查询由最后的 `* → PUBLIC-DOH` 兜底。Egern 的 `bootstrap` 使用直连 UDP，本配置保留 `223.5.5.5` 和 `119.29.29.29` 作为引导 DNS。局域网查询使用当前网络提供的系统 DNS。

</details>

<details>
<summary><strong>内置 13 个模块清单（默认启用 · 24h 自动更新）</strong></summary>

<br />

| 分类 | 模块 |
| :--- | :--- |
| **广告净化** | 去广告合集（`blockAds`）、微博去广告&净化、哔哩哔哩、百度贴吧、斗鱼直播、闲鱼、淘宝、拼多多、网易云音乐、懂球帝 |
| **功能增强** | YouTube 增强、iRingo WeatherKit、IP Attribution |

</details>

---

## Loon 配置

### 上手步骤

1. **导入配置**：在 Loon 中打开 **配置 → 从 URL 导入**，粘贴上方 `Loon.conf` 地址。
2. **添加订阅**：导入节点订阅并在「默认代理」中选择可用节点或策略组，切换为 **规则模式**。

> [!TIP]
> 从旧版更新时，请**关闭「全覆盖更新」**以保留本地订阅、节点与证书；更新后刷新远程规则与插件即可。

| 核心策略组 | 默认行为 |
| :--- | :--- |
| **默认代理** | Google、YouTube、Telegram 等海外服务默认跟随此组 |
| **AI** | 海外 AI 默认走「美国」；DeepSeek、豆包等国内 AI 按规则直连 |
| **Apple / Microsoft** | 默认直连，可按需在组内切代理 |
| **漏网之鱼** | 未命中规则的流量默认走「默认代理」 |

<details>
<summary><strong>Loon 选路、去广告与 STUN 说明</strong></summary>

<br />

- **地区与倍率**：港 / 日 / 美 / 新 / 台按名称自动筛选并故障转移，其余归入「其他节点」；「自动优选」自动过滤高倍节点，另提供「低倍率节点」「高倍率节点」手动挑线。
- **去广告**：域名广告由 `AdBlock` 拦截；B 站、YouTube 等 HTTPS 重写插件需在 iOS 安装并信任 Loon 证书。
- **语音 / 联机异常**：默认启用 `disable-stun = true`；若通话或联机异常，将其改为 `false` 即可。

</details>

---

## IP 归属地分流

覆盖微博、哔哩哔哩、知乎、抖音、小红书、微信、网易云音乐等 30+ 常见 App，共 **242 条规则**（193 条代理 / 28 条直连 / 21 条拒绝），基于 [SunsetMkt/anti-ip-attribution](https://github.com/SunsetMkt/anti-ip-attribution) 转换并补充。

| 客户端 | 独立导入地址 | 使用说明 |
| :--- | :--- | :--- |
| **Egern** | `https://raw.githubusercontent.com/amilokane/Loondrift/main/plugin/ip-attribution.yaml` | `Egern.yaml` 已内置（默认 `PROXY: Proxy`）；独立使用时可自定义 `PROXY` 参数 |
| **Loon** | `https://raw.githubusercontent.com/amilokane/Loondrift/main/plugin/ip-attribution.plugin` | 单独导入插件，并将参数 `PROXY` 绑定至目标策略组 |

---

## 上游致谢

| 用途 | 来源 |
| :--- | :--- |
| **Egern 规则** | [Valiumlove/MultiClientProxyCore](https://github.com/Valiumlove/MultiClientProxyCore)（[Lanxu Space](https://lanxu-space.pages.dev/)） / [Repcz/EgernRules](https://github.com/Repcz/EgernRules) |
| **Loon 规则** | [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Loon) |
| **AI & 广告补充** | [VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules) / [RiverFlowsInUUU/RFU-Rules](https://github.com/RiverFlowsInUUU/RFU-Rules) |
| **模块与插件** | [Egern Hub](https://egern.pages.dev/) / [IBL3ND/module](https://github.com/IBL3ND/module) / [可莉插件中心](https://hub.kelee.one/) |
| **IP 归属地 & 图标** | [SunsetMkt/anti-ip-attribution](https://github.com/SunsetMkt/anti-ip-attribution) / [AIsouler/MyClash](https://github.com/AIsouler/MyClash) |

---

<div align="center">
  <sub><code>plugin/</code> 目录下规则遵循上游 <a href="plugin/LICENSE">GPL-3.0</a> 协议 &nbsp;|&nbsp; 版本规则：每次更新小数位 +1，满 10 进位（<code>2.9</code> → <code>3.0</code>）</sub>
</div>

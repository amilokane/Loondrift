<p align="center">
  <img src="assets/loondrift-banner.jpg" alt="Loondrift" width="900">
</p>

<p align="center">
  <strong>为 iPhone 准备的 Loon 和 Egern 分流配置</strong><br>
  国内站点通常直连 · 海外 AI 独立选路 · 域名广告拦截
</p>

<p align="center">
  <a href="#egern-导入"><strong>Egern 导入</strong></a>　·　<a href="#快速导入">Loon 导入</a>　·　<a href="#常用策略">常用策略</a>
</p>

## Egern 导入

1. 在 Egern 首页从 URL 导入 [Egern.yaml](https://raw.githubusercontent.com/amilokane/Loondrift/main/Egern.yaml)。
2. 添加自己的 Egern 兼容订阅，选择 `Airport` 策略组。地区组和 `Smart` 会从这份订阅筛选节点。
3. `Proxy` 初始使用 `Smart`。ChatGPT、Claude 和 Google AI 共用 `AI` 策略组，默认选 `United States`。国内 AI 直连。如果订阅没有美国节点，可以在 `AI` 里切到 `Proxy`。

`Smart` 使用 Egern 的智能选优，综合延迟、抖动和可靠性，并过滤已识别的高倍率节点。[Egern 策略组文档](https://egernapp.com/zh-CN/docs/configuration/policy_groups#智能选优)

导入配置会一并加载远程模块。默认启用「广告拦截&净化合集」「微博去广告&净化」「苹果天气增强」，以及哔哩哔哩、斗鱼、闲鱼、YouTube、淘宝、拼多多、网易云音乐和懂球帝八个单项。合集也包含这八个 App 的规则。要单独测试某个 App 的净化效果，可先关闭合集，再切换对应单项。`AdBlock` 控制主配置的域名广告规则，各模块在 Egern 的模块列表中开关。[Egern Hub 模块目录](https://egern.pages.dev/)

需要 HTTPS 重写时，在 iOS 安装并信任自己的 Egern 证书，解密范围使用模块声明的域名。苹果天气增强默认将匹配的天气请求转交 `weatherkit.pages.dev`。IP 归属地模块默认关闭，开启后可在「IP 属地」策略组选择出口，初始跟随 `Proxy`。[Egern 模块文档](https://egernapp.com/zh-CN/docs/configuration/modules)

Egern 版的服务与广告规则由 Repcz 从 blackmatrix7 转换为原生 YAML。国内 AI 和 Google AI 使用 VPSDance 的规则，ChatGPT、Claude 和 Threads 使用 MetaCubeX 的远程列表。[Repcz/EgernRules](https://github.com/Repcz/EgernRules) · [VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules/tree/main/rules/egern) · [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat/tree/meta/geo/geosite/classical)

每次更新 Egern 配置时，同步递增[文件首行的版本号](Egern.yaml#L1)。

## 快速导入

在 iPhone 的 Loon 中打开「配置」，选择从 URL 导入，粘贴下面的地址。

```text
https://raw.githubusercontent.com/amilokane/Loondrift/main/Loon.conf
```

1. 添加自己的节点订阅，在「默认代理」中选一个可用节点或策略组。
2. 切换到规则模式。想让 AI、YouTube 等服务用不同线路，可以在各自的策略组里选。

> [!TIP]
> **从旧配置更新**　关闭「全覆盖更新」以保留手机上的节点、订阅和证书。更新后刷新远程规则和插件，再确认常用服务的策略选择。

## 常用策略

| 策略 | 初始行为 |
| :--- | :--- |
| **默认代理** | Google、YouTube、Telegram 等服务初始跟随此组。 |
| **AI** | ChatGPT、Claude、Gemini 等海外 AI 初始走「美国」。DeepSeek、豆包等国内 AI 按规则直连。 |
| **Apple 与 Microsoft** | 初始直连，可在各自的策略组切换。 |
| **漏网之鱼** | 处理未命中规则的请求，初始走「默认代理」。 |

<details>
<summary><strong>选地区或固定节点</strong></summary>

- **按地区选**　香港、日本、美国、新加坡和台湾根据节点名称筛选，并在组内故障转移。德国、英国、韩国等线路可在「其他节点」查找。
- **按测速选**　「自动优选」按测速选择线路，并过滤已识别的高倍率节点。
- **固定单个节点**　先在服务组里选「手动选择」，再进入该组指定节点。引用「手动选择」的服务会共用当前节点。

</details>

## 设备设置

<details>
<summary><strong>启用 App 内去广告</strong></summary>

域名广告拦截由「AdBlock」控制，初始为拦截。B 站、YouTube 等需要解密 HTTPS 的插件，还需要在 iOS 安装并信任 Loon 证书。MitM over HTTP/2 和 QUIC 回退保护可按插件说明设置。

</details>

<details>
<summary><strong>排查通话或联机问题</strong></summary>

配置默认禁用 STUN。如果语音、视频通话或联机遇到问题，可以将 `disable-stun` 改为 `false` 再试。

</details>

## 上游来源

- 国内外域名、服务分流和广告拦截。[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Loon)
- 国内和海外 AI。[VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules/tree/main/rules/loon)
- Threads。[MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat/blob/meta/geo/geosite/classical/threads.list)
- 去广告和增强插件。[可莉的插件中心](https://hub.kelee.one/)

配置版本见 [Loon.conf 首行](Loon.conf#L1)。

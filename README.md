# Loondrift

一份面向 iPhone 的 Loon 分流配置。国内站点通常直连，ChatGPT、YouTube、Telegram 等服务可以分别选线路。配置自带广告拦截和常用 App 去广告插件。

## 在 iPhone 上导入

1. 复制下方地址，在 Loon 的配置页面选择从 URL 导入。
2. 在 Loon 添加自己的节点订阅，然后在「默认代理」里选一条可用线路。
3. 使用规则模式，按需为 AI、视频或社交服务单独选线路。

```text
https://raw.githubusercontent.com/amilokane/Loondrift/main/Loon.conf
```

更新已有配置前，关闭「全覆盖更新」以保留设备上的节点、订阅和证书。更新后刷新远程规则和插件，再检查常用服务的策略选择。

## 日常分流

| 策略 | 怎么选 |
| --- | --- |
| 默认代理 | Google、YouTube、Telegram 等服务初始跟随此组。可选地区、自动优选或手动选择。 |
| AI | ChatGPT、Claude、Gemini 等海外 AI 服务初始走「美国」。如果订阅没有美国节点，可在 AI 组另选线路。DeepSeek、豆包等国内 AI 按国内规则直连。 |
| Apple、Microsoft | 初始直连。需要代理时，在对应服务组切换。 |
| 漏网之鱼 | 接管未命中规则的请求，初始跟随「默认代理」，也可切到直连。 |

香港、日本、美国、新加坡和台湾组按节点名称筛选线路，并在组内故障转移。德国、英国、韩国等线路可在「其他节点」中查找。「自动优选」按测速选择线路，并过滤已识别的高倍率节点。

要固定某项服务的出口，先在该服务组选「手动选择」，再进入「手动选择」指定节点。引用这个组的服务会共用当前选中的节点。

## 去广告与兼容性

- 域名广告拦截由「AdBlock」控制，初始为拦截。B 站、YouTube 等需要解密 HTTPS 的去广告插件，还需要在 iOS 安装并信任 Loon 证书。
- 在插件详情查看所需设置。涉及 MitM over HTTP/2 或 QUIC 回退保护时，按插件说明在 Loon 中开启。
- 配置默认禁用 STUN。如果语音、视频通话或联机遇到问题，可以将 `disable-stun` 改为 `false` 再试。

## 规则与插件来源

| 用途 | 上游 |
| --- | --- |
| 国内外域名、服务分流和广告拦截 | [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Loon) |
| 国内和海外 AI | [VPSDance/ai-proxy-rules](https://github.com/VPSDance/ai-proxy-rules/tree/main/rules/loon) |
| Threads | [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat/blob/meta/geo/geosite/classical/threads.list) |
| 去广告和增强插件 | [可莉的插件中心](https://hub.kelee.one/) |

配置版本见 [Loon.conf 首行](Loon.conf#L1)。

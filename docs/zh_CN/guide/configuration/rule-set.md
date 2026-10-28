# 规则集

当前唯一内置模板是 [`template/general.yaml`](https://github.com/realwuxiong/SubConv/blob/main/template/general.yaml)。公共规则统一使用 [blackmatrix7/ios_rule_script 的 Clash 规则](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Clash)；三个 Custom 列表由本仓库维护。

## 匹配顺序

`RULESET` 从上往下匹配，首条命中的规则决定使用的分组：

1. `CustomAI.list`、`CustomEmby.list`、`CustomProxy.list`，覆盖后面的直连、广告拦截及公共服务规则。如果多个 Custom 同时匹配，排在前面的生效。
2. GitHub 下载和 Google 更新的内联代理规则，然后是局域网和直连白名单，以及隐私、广告及应用净化规则。
3. AI、YouTube 等服务专用规则；AI 位于 Bing、Microsoft、Google 之前，YouTube 位于 Google 之前。
4. 国内/国外媒体汇总、通用代理、国内地址及下载规则。
5. `GEOIP,CN` 和最终兜底。

转换器会在模板规则之前添加自身域名的直连规则，确保订阅和规则更新地址可访问。现有分组名称、默认选择、三个手动选择组和地区节点组保持不变。

## 规则源映射

下表是 blackmatrix7 Clash 目录中的分类名，使用对应的 `<分类>/<分类>.list` 文件，格式为 `classical` 文本。

| 分组 | 分类 |
| --- | --- |
| 全球直连 | Lan、Direct、Adobe、ChinaMaxNoMedia、Download |
| 隐私防护 | Privacy |
| 广告拦截 | AdvertisingLite |
| 应用净化 | Advertising，作为前面隐私和精简广告规则的完整补充 |
| AI 平台 | OpenAI、Claude、Gemini、Copilot，加 CustomAI |
| 谷歌 FCM / 谷歌服务 | GoogleFCM / Google |
| 微软 Bing / 微软云盘 / 微软服务 | Bing / OneDrive / Microsoft |
| 苹果服务 / 电报消息 | Apple / Telegram |
| TikTok / Emby | TikTok / Emby，加 CustomEmby |
| 指定代理 | CustomProxy |
| 网易音乐 / Spotify | NetEaseMusic / Spotify |
| 游戏平台 | Epic、Origin、Sony、Steam、Nintendo |
| 油管视频 / 奈飞视频 / 巴哈姆特 | YouTube / Netflix / Bahamut |
| 哔哩哔哩 | BiliBiliIntl、BiliBili |
| 国内媒体 / 国外媒体 | ChinaMedia / GlobalMedia |
| 节点选择 | Proxy |

`ChinaMaxNoMedia` 包含国内域名、IPv4、IPv6 和其他规则，统一承担原来多份国内地址列表的分流职责；位于服务专用规则和通用代理规则之后。`BiliBiliIntl` 是国际版分类，与旧的港澳台番剧分类不完全等价，但仍使用原有哔哩哔哩分组。

blackmatrix7 部分 AI 分类不包含原先已覆盖的服务域名，因此 `CustomAI.list` 补充了 Claude 新域名、Cursor、NotebookLM、Perplexity、xAI、JetBrains AI 等，保留既有服务的 AI 分流。公共列表的内容和更新时间由各分类的上游决定，换源不代表每一份列表都每天发生变化。

## 刷新和生效

规则提供者保持每 7 天自动刷新，可以在客户端手动刷新。替换规则源后需要更新完整订阅配置；修改已有 Custom 文件时，在改动推送到仓库后刷新对应规则提供者即可。

本次迁移只更换规则数据与补充域名，DNS、端口、节点订阅更新间隔和 Geo 数据库配置保持原值。

规则按 `RULESET` 的顺序匹配。除了规则集 URL，也可以使用 `[]` 前缀添加内联规则，例如：

```yaml
RULESET:
- - 🚀 节点选择
  - '[]DOMAIN-SUFFIX,github.com'
```

内置 general 模板在三个 Custom 列表之后、公共规则之前将 `github.com`、`githubusercontent.com`（包括 GitHub Release 下载）以及 `update.googleapis.com`、`clientservices.googleapis.com` 交给「🚀 节点选择」，避免落入后面的直连规则。请为该组选择可用节点。

默认通过 SubConv 的 `/proxy` 转发规则集时，会过滤 mihomo 不支持的 `URL-REGEX` 行，并在服务端记录警告；其他规则（包括 `IP-ASN`）保留。使用 `notproxyrule` 让客户端直接下载规则时不会进行过滤，需要使用兼容 mihomo 的规则源。

更新服务后，请重新生成并更新订阅，并在客户端强制更新规则集缓存。OpenClash 的自定义覆写可能改变最终规则；如果仍出现 ASN 下载超时，请检查实际生效的 GitHub 分流，并重新更新 ASN 数据库。ASN 数据库与 `Country.mmdb` 不同。

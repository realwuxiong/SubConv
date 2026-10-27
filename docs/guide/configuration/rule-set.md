# Rule sets

The only bundled template is [`template/general.yaml`](https://github.com/realwuxiong/SubConv/blob/main/template/general.yaml). All public rule providers use the [blackmatrix7/ios_rule_script Clash rules](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Clash); this repository maintains the three Custom lists.

## Matching order

`RULESET` is evaluated from top to bottom; the first matching rule selects the routing group:

1. `CustomAI.list`, `CustomEmby.list`, and `CustomProxy.list` override later direct, blocking, and public service rules. If multiple Custom lists match, the earlier list wins.
2. LAN and direct exceptions, followed by privacy, advertising, and application filtering.
3. Service-specific lists: AI precedes Bing, Microsoft, and Google; YouTube precedes Google.
4. Domestic/global media aggregates, general proxy rules, domestic addresses, and downloads.
5. `GEOIP,CN` and the final fallback.

The converter adds its own hostname direct rule before the template rules to keep updates reachable. Existing group names, default choices, three manual selectors, and regional node groups are preserved.

## Provider mapping

Names below refer to categories in the blackmatrix7 Clash directory. Providers use `<category>/<category>.list` in classical text format.

| Routing group | Categories |
| --- | --- |
| 全球直连 | Lan, Direct, Adobe, ChinaMaxNoMedia, Download |
| 隐私防护 | Privacy |
| 广告拦截 | AdvertisingLite |
| 应用净化 | Advertising, the full supplement after privacy and lightweight advertising rules |
| Ai平台 | OpenAI, Claude, Gemini, Copilot, plus CustomAI |
| 谷歌FCM / 谷歌服务 | GoogleFCM / Google |
| 微软Bing / 微软云盘 / 微软服务 | Bing / OneDrive / Microsoft |
| 苹果服务 / 电报消息 | Apple / Telegram |
| TikTok / Emby | TikTok / Emby, plus CustomEmby |
| 指定代理 | CustomProxy |
| 网易音乐 / Spotify | NetEaseMusic / Spotify |
| 游戏平台 | Epic, Origin, Sony, Steam, Nintendo |
| 油管视频 / 奈飞视频 / 巴哈姆特 | YouTube / Netflix / Bahamut |
| 哔哩哔哩 | BiliBiliIntl, BiliBili |
| 国内媒体 / 国外媒体 | ChinaMedia / GlobalMedia |
| 节点选择 | Proxy |

`ChinaMaxNoMedia` includes domestic domains, IPv4, IPv6, and other rule types, replacing several domestic address lists. It follows service-specific and general proxy rules. `BiliBiliIntl` targets the international service and is not an exact replacement for the old Hong Kong/Macau/Taiwan anime category; it still uses the existing Bilibili group.

Some blackmatrix7 AI categories omit previously covered service domains. `CustomAI.list` supplements newer Claude domains, Cursor, NotebookLM, Perplexity, xAI, JetBrains AI, and other services to preserve their AI routing. Each upstream category determines its own contents and update date; changing providers does not mean every list changes daily.

## Refreshing and applying changes

Rule providers still refresh automatically every 7 days and can be refreshed manually. Update the complete subscription config after this source migration. For changes to an existing Custom file, push the changes to the repository and refresh that rule provider.

This migration changes rule data and supplemental domains; DNS, ports, subscription update intervals, and Geo database settings retain their existing values.

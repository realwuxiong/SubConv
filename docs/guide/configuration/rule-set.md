# Rule sets

The only bundled template is [`template/general.yaml`](https://github.com/realwuxiong/SubConv/blob/main/template/general.yaml). Its `RULESET` entries are evaluated in order; the first matching rule decides the routing group.

`CustomAI.list`, `CustomEmby.list`, and `CustomProxy.list` come first so your custom rules override the later built-in rules, including ad blocking and direct routing. If custom lists overlap, their order above decides which group wins.

The AI and YouTube rule sets precede the general Google rule set so their traffic reaches the dedicated groups. General proxy rules, domestic direct rules, `GEOIP,CN`, and the final fallback follow the service-specific rules.

The converter adds a direct rule for its own hostname before the template rules so subscription and rule updates remain reachable.

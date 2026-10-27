# 规则集

当前唯一内置模板是 [`template/general.yaml`](https://github.com/realwuxiong/SubConv/blob/main/template/general.yaml)。其中的 `RULESET` 按顺序匹配，首条命中的规则决定流量进入哪个分组。

`CustomAI.list`、`CustomEmby.list`、`CustomProxy.list` 排在最前面，因此你的自定义规则会覆盖后面的内置规则，包括广告拦截和直连规则。如果自定义列表之间有重叠，则按上述顺序决定使用哪个分组。

AI 和 YouTube 专用规则排在 Google 通用规则之前，确保相应流量进入专用分组。通用代理规则、国内直连规则、`GEOIP,CN` 和最终兜底规则排在服务专用规则之后。

转换器会在模板规则之前添加自身域名的直连规则，确保订阅和规则更新地址可访问。

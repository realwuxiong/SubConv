# SubConv

[English](README.md) | 中文

![license](https://img.shields.io/github/license/realwuxiong/SubConv)
![last commit](https://img.shields.io/github/last-commit/realwuxiong/SubConv)

SubConv 用于将订阅链接或 Clash YAML 转换为 mihomo 兼容配置。

这个 fork 将后端、Web UI 和文档放在同一个仓库中维护。默认运行时模板是 `general`，也内置了 `zju` 模板。

## 功能

- 支持 Clash YAML 订阅和常见 V2Ray 分享链接转换。
- 支持从一个或多个订阅源生成 mihomo 配置。
- 使用仓库根目录 `template/` 下的模板文件。
- 提供 Vue Web UI，方便在浏览器中转换。
- 提供订阅转换、proxy-provider 和规则代理相关 API。

## 快速开始

后端：

```bash
uv sync
cp config.yaml.example config.yaml
uv run python api.py
```

前端：

```bash
cd mainpage
bun install
bun run dev
```

文档：

```bash
cd docs
bun install
bun run dev
```

## 配置

运行时配置按以下顺序加载：

1. 环境变量
2. `config.yaml`
3. `config.yaml.example`

重要文件：

- `config.yaml.example`：默认运行时配置
- `template/general.yaml`：默认 mihomo 模板
- `template/zju.yaml`：额外内置模板

## 本 fork 的规则分组

默认的 `general` 模板增加了以下可独立选择节点的分组：

- `🎵 TikTok`：使用持续维护的 [blackmatrix7 TikTok 规则集](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/TikTok/TikTok.list)。
- `🍿 Emby`：同时使用 [blackmatrix7 Emby 规则集](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/Emby/Emby.list) 和仓库内的 [`rules/CustomEmby.list`](rules/CustomEmby.list)，目前包含 `uhdnow.com` 及其所有子域名。
- `📍 指定代理`：使用仓库内的 [`rules/CustomProxy.list`](rules/CustomProxy.list)，其中的网站必须使用代理；目前包含 `nodeseek.com` 及其所有子域名。

两个本地规则文件均使用 mihomo classical 文本格式，每行添加一条规则，例如：

```text
DOMAIN,www.example.com
DOMAIN-SUFFIX,example.com
DOMAIN-KEYWORD,example
```

私人 Emby 服务器域名添加到 `rules/CustomEmby.list`，其他需要独立选择代理的网站添加到 `rules/CustomProxy.list`。修改推送到仓库后，在 mihomo 中刷新对应的规则提供者即可生效。

## API

- `/sub`：将订阅源转换为完整 mihomo 配置
- `/provider`：将订阅源转换为 proxy-provider 响应
- `/proxy`：代理所选模板允许的 rule-provider URL
- `/config`：提供前端运行时选项

## 项目结构

- `subconv/`：FastAPI 后端与转换逻辑
- `api.py`：本地运行和 Vercel 使用的后端入口
- `mainpage/`：Vue/Vite 前端
- `docs/`：VitePress 文档
- `template/`：运行时 mihomo 模板
- `rules/`：由本仓库维护的自定义 rule-provider 文件

## 许可证

本项目采用 [MPL-2.0 License](LICENSE) 分发。

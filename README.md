# SubConv

English | [中文](README_CN.md)

![license](https://img.shields.io/github/license/realwuxiong/SubConv)
![last commit](https://img.shields.io/github/last-commit/realwuxiong/SubConv)

SubConv converts subscription links or Clash YAML into mihomo-compatible configuration files.

This fork keeps the backend, web UI, and documentation in one repository. The only built-in runtime template is `general`.

## Features

- Convert Clash YAML subscriptions and common V2Ray share links.
- Generate mihomo configs from one or more subscription sources.
- Use template files from the root `template/` directory.
- Serve a Vue web UI for browser-based conversion.
- Provide API endpoints for subscription, proxy-provider, and rule proxy usage.

## Quick Start

Backend:

```bash
uv sync
cp config.yaml.example config.yaml
uv run python api.py
```

Frontend:

```bash
cd mainpage
bun install
bun run dev
```

Docs:

```bash
cd docs
bun install
bun run dev
```

## Configuration

Runtime settings are loaded in this order:

1. Environment variables
2. `config.yaml`
3. `config.yaml.example`

Important files:

- `config.yaml.example`: default runtime settings
- `template/general.yaml`: default mihomo template

Public rules in the bundled `general` template come from [blackmatrix7/ios_rule_script, Clash directory](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Clash); this repository maintains the custom rules. See the [rule-set mapping](docs/guide/configuration/rule-set.md). All rule providers still refresh every 7 days and can also be refreshed manually in the client.

## Fork-specific rule groups

The default `general` template adds these independently selectable groups:

- `🚀 手动切换1` / `🚀 手动切换2` / `🚀 手动切换3`: three manual selectors that include primary and standby subscription nodes.
- `🎵 TikTok`: uses the actively maintained [blackmatrix7 TikTok ruleset](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/TikTok/TikTok.list).
- `💬 Ai平台`: uses blackmatrix7 OpenAI, Claude, Gemini, and Copilot categories, supplemented by [`rules/CustomAI.list`](rules/CustomAI.list) for Muse, Meta AI, Cursor, NotebookLM, and other service domains.
- `🍿 Emby`: combines the [blackmatrix7 Emby ruleset](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/Emby/Emby.list) with [`rules/CustomEmby.list`](rules/CustomEmby.list), which currently includes `uhdnow.com` and all of its subdomains.
- `📍 指定代理` (designated proxy): uses [`rules/CustomProxy.list`](rules/CustomProxy.list). Sites in this file are required to use a proxy; it currently includes `nodeseek.com` and all of its subdomains.
- `🎶 Spotify` defaults to `🚀 节点选择`, while the final `🐟 漏网之鱼` fallback defaults to `DIRECT`.

Custom rules are matched first in this order: `CustomAI.list`, `CustomEmby.list`, then `CustomProxy.list`. They override later built-in rules, including ad blocking and direct routing. AI and YouTube rules precede the general Google rules. The converter keeps its own hostname direct rule ahead of the template rules.

All three custom ruleset files use mihomo classical text format. Add one rule per line, for example:

```text
DOMAIN,www.example.com
DOMAIN-SUFFIX,example.com
DOMAIN-KEYWORD,example
```

Use `rules/CustomAI.list` for AI sites, `rules/CustomEmby.list` for private Emby servers, and `rules/CustomProxy.list` for other sites that need a dedicated proxy choice. Changes become available after they are pushed to the repository and the rule provider is refreshed in mihomo.

## API

- `/sub`: convert subscription sources into a full mihomo config
- `/provider`: convert subscription sources into a proxy-provider response
- `/proxy`: proxy rule-provider URLs allowed by the selected template
- `/config`: expose frontend runtime options

## Project Layout

- `subconv/`: FastAPI backend and converter logic
- `api.py`: backend entrypoint for local runtime and Vercel
- `mainpage/`: Vue/Vite frontend
- `docs/`: VitePress documentation
- `template/`: runtime mihomo templates
- `rules/`: repository-maintained custom rule-provider files

## License

This project is distributed under the [MPL-2.0 License](LICENSE).

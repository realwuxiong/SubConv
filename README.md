# SubConv

English | [中文](README_CN.md)

![license](https://img.shields.io/github/license/realwuxiong/SubConv)
![last commit](https://img.shields.io/github/last-commit/realwuxiong/SubConv)

SubConv converts subscription links or Clash YAML into mihomo-compatible configuration files.

This fork keeps the backend, web UI, and documentation in one repository. The default runtime template is `general`; `zju` is also available.

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
- `template/zju.yaml`: additional built-in template

## Fork-specific rule groups

The default `general` template adds these independently selectable groups:

- `🎵 TikTok`: uses the actively maintained [blackmatrix7 TikTok ruleset](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/TikTok/TikTok.list).
- `🍿 Emby`: combines the [blackmatrix7 Emby ruleset](https://github.com/blackmatrix7/ios_rule_script/blob/master/rule/Clash/Emby/Emby.list) with [`rules/CustomEmby.list`](rules/CustomEmby.list), which currently includes `uhdnow.com` and all of its subdomains.
- `📍 指定代理` (designated proxy): uses [`rules/CustomProxy.list`](rules/CustomProxy.list). Sites in this file are required to use a proxy; it currently includes `nodeseek.com` and all of its subdomains.

Both local ruleset files use mihomo classical text format. Add one rule per line, for example:

```text
DOMAIN,www.example.com
DOMAIN-SUFFIX,example.com
DOMAIN-KEYWORD,example
```

Use `rules/CustomEmby.list` for private Emby servers and `rules/CustomProxy.list` for other sites that need a dedicated proxy choice. Changes become available after they are pushed to the repository and the rule provider is refreshed in mihomo.

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

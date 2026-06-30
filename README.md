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

## License

This project is distributed under the [MPL-2.0 License](LICENSE).

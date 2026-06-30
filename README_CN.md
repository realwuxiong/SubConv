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

## 许可证

本项目采用 [MPL-2.0 License](LICENSE) 分发。

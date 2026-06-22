# 第 02 课：安装、依赖与运行环境

## 学习目标

- 理解项目依赖为什么按 core、ui、all-frontends 分层。
- 掌握 Python 版本限制和常见安装方式。
- 能区分“人类手动安装”和“Agent 自己补齐能力”的边界。
- 能启动至少一个前端入口并定位启动失败原因。

## 源码地图

- `pyproject.toml`：依赖声明与可选 extras。
- `docs/installation.md`：英文安装说明。
- `docs/installation_zh.md`：中文安装说明。
- `docs/GETTING_STARTED.md`：新手启动路径。
- `assets/ga_install.sh`、`assets/ga_install.ps1`：一键安装脚本。
- `launch.pyw`：Streamlit 启动入口。
- `frontends/tui_v3.py`：推荐终端入口。

## 核心知识点

### 1. Python 版本约束

`pyproject.toml` 中写明：

```toml
requires-python = ">=3.10,<3.14"
```

实际学习和运行建议使用 Python 3.11 或 3.12。原因是部分 UI 依赖，尤其是 `pywebview`，可能不支持太新的 Python。

### 2. 依赖分层

核心依赖很少：

- `requests`
- `beautifulsoup4`
- `bottle`
- `simple-websocket-server`
- `aiohttp`

这些支撑 LLM 请求、本地 WebDriver 服务、HTML 解析和本地 HTTP/WebSocket 服务。

可选依赖：

- `.[ui]`：Streamlit、TUI、桌面 UI 相关。
- `.[all-frontends]`：Telegram、QQ、飞书、企业微信、钉钉等 IM 前端。

项目故意不默认安装所有能力，因为 GenericAgent 的哲学是“需要时再让 Agent 自己补能力”。

### 3. 启动入口

推荐入口：

```bash
python frontends/tui_v3.py
```

Web UI：

```bash
python launch.pyw
```

核心 SDK 方式：

```python
from agentmain import GenericAgent

agent = GenericAgent()
```

## 精读步骤

1. 读 `pyproject.toml`，把 dependencies 和 optional-dependencies 分开记录。
2. 读 `docs/GETTING_STARTED.md` 的 Python、API Key、初次启动章节。
3. 对照 `README.md` Quick Start，比较两种文档面对的读者差异。
4. 打开 `launch.pyw`，确认它只是前端入口，不是核心 Agent。
5. 打开 `frontends/tui_v3.py` 顶部导入，找到它如何引入 `GeneraticAgent`。

## 动手练习

### 练习 1：检查当前环境

运行：

```bash
python --version
python -c "import sys; print(sys.executable)"
```

记录：

- Python 版本；
- 解释器路径；
- 是否满足 `<3.14`。

### 练习 2：理解 editable install

阅读并解释：

```bash
uv pip install -e ".[ui]"
```

需要说明：

- `-e` 的含义；
- `.[ui]` 的含义；
- 为什么开发期适合 editable install。

### 练习 3：启动 TUI

运行：

```bash
python frontends/tui_v3.py
```

如果失败，按顺序检查：

1. 是否缺少 UI 依赖；
2. 是否缺少 `mykey.py`；
3. 是否 Python 版本不兼容；
4. 是否当前目录不是项目根目录。

## 常见问题

### 为什么不一次装所有依赖？

因为 IM、桌面、Web 自动化、OCR、视觉等能力依赖很杂。一次装全会：

- 增加安装失败概率；
- 污染环境；
- 降低跨平台稳定性；
- 违背“能力按需生长”的设计。

### 为什么项目可以在缺依赖时继续存在？

很多能力是可选前端或可选 SOP。核心 Agent Loop 和工具层不依赖所有 UI 包。

## 检查清单

- [ ] 能解释 `dependencies` 和 `optional-dependencies` 的区别。
- [ ] 能说明为什么推荐 Python 3.11/3.12。
- [ ] 能启动 TUI 或知道失败卡在哪一步。
- [ ] 能指出哪些能力属于 core，哪些属于 optional frontend。
- [ ] 能独立阅读安装文档并整理一份最小安装命令。


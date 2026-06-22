# 第 17 课：前端、桌面端与插件扩展

## 学习目标

- 理解多个前端如何复用同一个 Agent 核心。
- 掌握 TUI、Streamlit、IM、桌面端的差异。
- 理解桌面桥接层的 HTTP/WebSocket API。
- 掌握插件 hook 系统和 Langfuse tracing 的扩展方式。

## 源码地图

- `frontends/tui_v3.py`
- `frontends/stapp.py`
- `frontends/stapp2.py`
- `frontends/chatapp_common.py`
- `frontends/tgapp.py`
- `frontends/dcapp.py`
- `frontends/fsapp.py`
- `frontends/desktop_bridge.py`
- `frontends/desktop/`
- `frontends/desktop/src-tauri/src/lib.rs`
- `plugins/hooks.py`
- `plugins/project_mode.py`
- `plugins/langfuse_tracing.py`

## 前端分层

```text
用户界面
-> 前端适配层
-> GenericAgent.put_task()
-> display_queue
-> 前端渲染输出
```

不同前端差异主要在：

- 输入方式；
- 输出渲染；
- 会话管理；
- 文件和图片处理；
- 中断控制；
- slash 命令；
- 部署环境。

## TUI

`frontends/tui_v3.py` 是推荐入口，特点：

- prompt_toolkit 输入；
- rich 渲染；
- slash 命令；
- 会话恢复；
- 模型切换；
- token 成本展示；
- 多语言 UI；
- 支持文件和图片占位符。

学习重点：

- 如何创建 `GeneraticAgent`；
- 如何调用 `put_task()`；
- 如何消费输出队列；
- slash 命令如何映射到功能模块。

## Streamlit 与其他 Python UI

`frontends/stapp.py`、`stapp2.py` 提供 Web UI。它们适合学习：

- 会话状态；
- 页面渲染；
- 长任务刷新；
- Python UI 与后台线程配合。

## IM 前端

IM 前端包括：

- Telegram：`tgapp.py`
- Discord：`dcapp.py`
- 飞书：`fsapp.py`
- QQ、微信、企业微信、钉钉等。

共性逻辑在 `chatapp_common.py` 中。学习重点：

- 消息事件如何转成 prompt；
- 用户身份和会话如何映射；
- 图片、文件如何处理；
- 回复如何分段发送。

## 桌面端

桌面端由两部分组成：

```text
Tauri shell
-> Rust lib.rs
-> desktop_bridge.py
-> GenericAgent
```

`desktop_bridge.py` 提供 HTTP API：

- `/status`
- `/config`
- `/model-profiles`
- `/sessions`
- `/session/new`
- `/session/{sid}/prompt`
- `/session/{sid}/messages`
- `/session/{sid}/cancel`

WebSocket 只推送轻量 session-state 事件。

Rust 侧负责：

- 查找 Python；
- 查找项目目录；
- 启动 bridge；
- 等待端口；
- 保存桌面配置。

## 插件系统

`plugins/hooks.py` 提供简单 hook 注册：

- `agent_before`
- `turn_before`
- `llm_before`
- `llm_after`
- `tool_before`
- `tool_after`
- `turn_after`
- `agent_after`

插件通过：

```python
@register("event_name")
def callback(ctx):
    ...
```

接入主流程。

`agentmain.py` 启动时会：

```python
from plugins.hooks import discover_and_load
discover_and_load()
```

因此 `plugins/` 下非 `_` 开头的 `.py` 文件会被自动加载。

## 精读步骤

1. 读 `frontends/tui_v3.py` 顶部结构和 `GeneraticAgent` 使用点。
2. 搜索 `put_task`，看 TUI 如何提交任务。
3. 读 `frontends/chatapp_common.py`，总结 IM 前端通用抽象。
4. 读 `frontends/desktop_bridge.py` 的 `AgentManager`。
5. 读 `frontends/desktop/src-tauri/src/lib.rs`，理解 Rust 如何启动 Python bridge。
6. 读 `plugins/hooks.py`。
7. 读 `plugins/langfuse_tracing.py`，看 hook 如何做观测。

## 动手练习

### 练习 1：前端对比表

填写：

| 前端 | 输入来源 | 输出方式 | 会话管理 | 适用场景 |
|---|---|---|---|---|
| TUI | 本地终端 | rich/prompt_toolkit | 本地 session | 开发者使用 |
| Streamlit | 浏览器 | Web 页面 | Streamlit state | 快速 Web UI |
| IM | 聊天消息 | 平台消息 | 用户/群映射 | 远程协作 |
| Desktop | 桌面窗口 | 静态 Web + bridge | HTTP sessions | 普通用户 |

### 练习 2：设计一个插件

设计一个 `plugins/simple_logger.py`：

- 在 `tool_before` 记录工具名；
- 在 `tool_after` 记录执行结果状态；
- 不改变 ctx。

说明它为什么不应该吞异常或修改主流程。

### 练习 3：桌面端请求流程

画出：

```text
用户在桌面 UI 输入
-> POST /session/{sid}/prompt
-> AgentManager.submit_prompt
-> run_agent_turn
-> GenericAgent.put_task
-> messages API 轮询
```

## 常见误区

### 误区 1：每个前端都有自己的 Agent 实现

不是。前端应尽量只做输入输出适配，核心逻辑集中在 `GenericAgent`。

### 误区 2：插件应该随意改 ctx

插件可以返回 dict 修改 ctx，但这会影响主流程。除非非常明确，否则观测类插件应只读。

## 检查清单

- [ ] 能解释多个前端如何复用 `GenericAgent`。
- [ ] 能说出 TUI 的主要功能模块。
- [ ] 能说明 IM 前端的通用抽象。
- [ ] 能画出桌面端 Tauri 到 Python bridge 的链路。
- [ ] 能写出一个只读 hook 插件的设计。


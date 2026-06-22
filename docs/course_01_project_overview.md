# 第 01 课：项目全景与设计哲学

## 学习目标

- 理解 GenericAgent 要解决的问题：用极小代码把 LLM 接入本地电脑能力。
- 建立项目地图，知道核心代码、前端、记忆、插件和文档分别在哪里。
- 理解“最小内核 + 原子工具 + 自演化 Skill”的设计取舍。
- 能用自己的话解释为什么本项目没有引入 LangChain、Playwright 等重型框架。

## 先修知识

- Python 基础：模块导入、类、生成器、线程、队列。
- LLM 基础：系统提示词、上下文、工具调用、流式输出。
- 命令行基础：能读 `python xxx.py`、`pip/uv`、环境变量。

## 源码地图

- `README.md`：项目定位、能力说明、架构概览。
- `pyproject.toml`：依赖分层，区分 core、ui、all-frontends。
- `agent_loop.py`：核心 Agent Loop。
- `ga.py`：9 个原子工具的主要实现。
- `llmcore.py`：多模型适配、上下文压缩、工具调用协议。
- `agentmain.py`：Agent 对外入口和任务队列。
- `memory/`：分层记忆、SOP、能力沉淀。
- `frontends/`：TUI、Streamlit、IM、桌面桥接等入口。
- `plugins/`：Hook 插件系统。
- `reflect/`：自主运行、目标模式、定时任务等反射机制。

## 核心概念

### 1. Seed Code

GenericAgent 的核心思想不是把所有能力预装进代码，而是只保留足够小的“种子代码”。种子代码负责：

- 接收用户任务；
- 调用 LLM；
- 暴露少量原子工具；
- 执行工具结果；
- 把经验写入记忆。

因此，项目学习重点不是“有哪些内置业务功能”，而是“这些功能如何被 Agent 自己组合出来”。

### 2. 原子工具

`assets/tools_schema.json` 中定义的工具很少，包括：

- `code_run`
- `file_read`
- `file_patch`
- `file_write`
- `web_scan`
- `web_execute_js`
- `update_working_checkpoint`
- `ask_user`
- `start_long_term_update`

这些工具不是业务 API，而是电脑操作原语。Agent 用它们组合出文件编辑、网页自动化、长期记忆、调试、规划等高级能力。

### 3. 自演化 Skill

项目把可复用经验沉淀在 `memory/` 中。一个任务完成后，Agent 可以把“以后还会用到的步骤和坑点”写成 SOP。长期使用后，能力树不是由开发者一次性写完，而是从真实任务中长出来。

### 4. 多入口但单核心

项目有很多前端：

- 终端 UI：`frontends/tui_v3.py`
- Streamlit：`launch.pyw`、`frontends/stapp.py`
- IM：Telegram、Discord、飞书、QQ、微信等
- 桌面端：`frontends/desktop_bridge.py` + Tauri

但它们最终都围绕 `agentmain.GenericAgent` 运转。

## 精读路线

1. 读 `README.md` 的 Overview、Architecture、Self-Evolution 章节。
2. 打开 `pyproject.toml`，确认核心依赖只有少量包。
3. 用 `rg --files` 看项目目录，不急着读全部实现。
4. 先读 `agent_loop.py`，感受主循环规模。
5. 再读 `ga.py` 的 `GenericAgentHandler`，确认工具是如何挂载的。
6. 最后回到 `README.md`，重新理解“minimal”的含义。

## 动手练习

### 练习 1：画项目分层图

把项目画成 5 层：

1. 用户入口层：TUI、Web、IM、Desktop。
2. Agent 入口层：`GenericAgent`。
3. Agent Loop 层：`agent_runner_loop`。
4. 工具层：`GenericAgentHandler.do_*`。
5. 外部世界层：文件系统、终端、浏览器、LLM API、记忆文件。

### 练习 2：列出最短调用链

从“用户在 TUI 输入一句话”开始，写出调用链：

```text
frontends/tui_v3.py
-> agentmain.GenericAgent.put_task()
-> agentmain.GenericAgent.run()
-> agent_loop.agent_runner_loop()
-> llmcore.*Client.chat()
-> ga.GenericAgentHandler.dispatch()
```

### 练习 3：解释为什么核心工具少

写一段 300 字说明：

- 工具少带来的好处；
- 工具少带来的风险；
- 为什么 SOP 和记忆可以弥补一部分风险。

## 检查清单

- [ ] 能说清 GenericAgent 的核心目标。
- [ ] 能指出 Agent Loop、工具、模型适配、记忆分别在哪些文件。
- [ ] 能解释原子工具和业务工具的区别。
- [ ] 能解释为什么前端很多但核心入口统一。
- [ ] 能用一张图描述项目总体架构。


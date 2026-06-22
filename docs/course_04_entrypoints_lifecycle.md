# 第 04 课：程序入口与 Agent 生命周期

## 学习目标

- 理解 `GenericAgent` 如何接收任务、排队、执行、输出。
- 掌握前端与核心 Agent 的连接方式。
- 理解任务中断、长输入落盘、工作记忆传递等生命周期细节。
- 能追踪一次完整任务从输入到最终回答的过程。

## 源码地图

- `agentmain.py`：核心入口。
- `agent_loop.py`：每轮推理和工具执行。
- `ga.py`：工具处理器。
- `frontends/tui_v3.py`：TUI 如何调用 `put_task()`。
- `frontends/desktop_bridge.py`：HTTP/WebSocket 桥接如何管理会话。

## 生命周期总览

```text
用户输入
-> 前端调用 GenericAgent.put_task()
-> task_queue 入队
-> GenericAgent.run() 后台线程取任务
-> 构造 system prompt
-> 创建 GenericAgentHandler
-> 调用 agent_runner_loop()
-> LLM 返回文本或工具调用
-> handler.dispatch() 执行工具
-> display_queue 输出流式片段
-> 前端展示结果
```

## 核心知识点

### 1. 任务队列

`GenericAgent.put_task()` 不直接运行任务，而是把任务放入 `self.task_queue`。这样前端可以：

- 快速返回一个输出队列；
- 后台线程持续消费；
- 支持任务运行期间流式更新。

### 2. 输出队列

每个任务都有自己的 `display_queue`。Agent 会放入：

- `next`：增量输出或阶段性输出；
- `done`：最终输出；
- `turn`：当前轮次；
- `outputs`：最近几轮输出片段。

### 3. System Prompt 构造

`get_system_prompt()` 会合并：

- `assets/sys_prompt*.txt`
- 当前日期；
- `get_global_memory()` 生成的记忆摘要。

这意味着每次任务都不是空白启动，而是带着项目的长期记忆索引。

### 4. Handler 生命周期

每个任务会创建新的 `GenericAgentHandler`。但模型 backend 的 history 可能继续保留，因此：

- handler 管工具执行和工作记忆；
- backend 管模型对话历史；
- `history_info` 管压缩后的任务摘要。

### 5. 中断机制

`abort()` 会：

- 设置 `stop_sig`；
- 向当前 handler 的 `code_stop_signal` 写入信号；
- 让长时间执行的 `code_run` 有机会终止。

## 精读步骤

1. 读 `GenericAgent.__init__()`，记录初始化字段。
2. 读 `load_llm_sessions()`，理解模型客户端何时加载。
3. 读 `put_task()`，确认前端拿到的是输出队列。
4. 读 `run()` 主循环，按顺序标注每一步。
5. 读 `_handle_slash_cmd()`，理解少量 slash 命令如何在核心层处理。
6. 对照 `frontends/tui_v3.py` 搜索 `put_task`，看前端如何消费队列。

## 动手练习

### 练习 1：手写最小 SDK 调用

写一个最小脚本，创建 `GenericAgent`，启动后台线程，提交任务并读取输出队列。

重点不是实际跑通模型，而是理解队列模型。

### 练习 2：追踪长输入处理

阅读 `run()` 中这段逻辑：

```python
if len(raw_query) > 2000:
    task_file = ...
    raw_query = f'Long user prompt saved to {task_file}. Read and execute.'
```

回答：

- 为什么长 prompt 要落盘？
- 这会改变 LLM 看到的任务吗？
- 对工具调用有什么影响？

### 练习 3：解释恢复会话

阅读 `/resume` 对应逻辑，说明它为什么让 Agent 自己读取 `temp/model_responses/`。

## 检查清单

- [ ] 能画出一次任务的生命周期。
- [ ] 能解释 task_queue 和 display_queue 的区别。
- [ ] 能指出 system prompt 的组成。
- [ ] 能解释 handler 与 backend history 的关系。
- [ ] 能说明任务中断如何传递到代码执行器。


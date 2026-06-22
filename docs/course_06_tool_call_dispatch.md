# 第 06 课：Tool Call 调度机制

## 学习目标

- 理解工具 schema、模型输出、handler 方法之间的对应关系。
- 掌握工具调用从 JSON 到 Python 方法执行的全过程。
- 理解多工具调用、工具结果合并和下一轮 prompt 的生成。
- 能新增一个简单工具并知道要改哪些位置。

## 源码地图

- `assets/tools_schema.json`
- `assets/tools_schema_cn.json`
- `agent_loop.py::BaseHandler.dispatch`
- `ga.py::GenericAgentHandler.do_*`
- `llmcore.py::ToolClient._parse_mixed_response`
- `llmcore.py::NativeToolClient.chat`

## 工具调用链

```text
tools_schema.json
-> client.chat(..., tools=tools_schema)
-> LLM 产生 tool call
-> MockResponse.tool_calls
-> agent_runner_loop()
-> handler.dispatch()
-> GenericAgentHandler.do_xxx()
-> StepOutcome
-> tool_results
-> 下一轮 prompt
```

## 核心知识点

### 1. schema 是给模型看的契约

`assets/tools_schema.json` 定义工具名、描述和参数结构。它不是执行代码，执行代码在 `ga.py`。

学习时要对照两边：

- schema 中的 `name`；
- handler 中的 `do_<name>`。

例如：

```text
file_read -> do_file_read
code_run -> do_code_run
web_execute_js -> do_web_execute_js
```

### 2. dispatch 的动态分发

`BaseHandler.dispatch()` 根据工具名拼出方法名：

```python
method_name = f"do_{tool_name}"
```

如果方法存在，就调用；否则返回“未知工具”。

### 3. 多工具调用

同一轮中可能有多个工具调用。主循环会给 args 注入：

- `_index`
- `_tool_num`

工具可以据此控制输出长度和是否跳过 anchor prompt。

### 4. 工具结果回传

工具的 `StepOutcome.data` 会序列化成字符串，作为 `tool_results` 回传给模型。

下一轮模型看到的是：

- 工具结果；
- 工作记忆；
- 历史摘要；
- handler 追加的系统提示。

## 精读步骤

1. 读 `assets/tools_schema.json`，列出全部工具。
2. 在 `ga.py` 中搜索 `def do_`，确认每个 schema 都有实现。
3. 读 `BaseHandler.dispatch()`。
4. 读 `agent_runner_loop()` 中 `tool_calls` 的构造。
5. 读 `ToolClient._parse_mixed_response()`，理解文本协议如何被解析成工具调用。
6. 读 `NativeToolClient.chat()`，理解原生 tool_result 如何对齐。

## 动手练习

### 练习 1：工具映射表

创建一张表：

| schema 工具名 | handler 方法 | 是否读写文件 | 是否可能长时间运行 |
|---|---|---|---|

至少填完 9 个工具。

### 练习 2：设计一个只读工具

设想新增 `env_info` 工具，用于返回 Python 版本和当前工作目录。

写出需要改动：

1. `assets/tools_schema.json` 添加 schema；
2. `ga.py` 添加 `do_env_info()`；
3. 如需中文模型支持，同步 `assets/tools_schema_cn.json`。

不要求真的提交代码，重点是理解扩展边界。

### 练习 3：分析未知工具

阅读 `dispatch()` 中未知工具处理逻辑，回答：

- 为什么要让 `client.last_tools = ''`？
- 这和工具描述缓存有什么关系？

## 检查清单

- [ ] 能解释 schema 和实现的关系。
- [ ] 能说出 `do_` 动态分发规则。
- [ ] 能理解 `_index` 和 `_tool_num` 的用途。
- [ ] 能追踪 tool_result 如何进入下一轮。
- [ ] 能列出新增工具需要改的文件。


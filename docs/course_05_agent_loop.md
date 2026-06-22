# 第 05 课：100 行 Agent Loop 深入解析

## 学习目标

- 完整读懂 `agent_runner_loop()`。
- 理解一轮 Agent 推理包含哪些阶段。
- 掌握 `StepOutcome` 如何控制继续、结束或中断。
- 能解释为什么主循环可以很短却能支撑复杂任务。

## 源码地图

- `agent_loop.py`
- `ga.py::GenericAgentHandler`
- `llmcore.py::ToolClient`
- `llmcore.py::NativeToolClient`

## Agent Loop 的职责

`agent_runner_loop()` 只负责编排，不直接实现具体工具。它做的事情是：

1. 构造初始 messages；
2. 调用 LLM；
3. 解析工具调用；
4. 分发工具；
5. 收集工具结果；
6. 生成下一轮 prompt；
7. 判断退出条件。

这种设计把“循环控制”和“工具行为”分离。

## 核心数据结构

### StepOutcome

```python
@dataclass
class StepOutcome:
    data: Any
    next_prompt: Optional[str] = None
    should_exit: bool = False
```

含义：

- `data`：工具返回给模型的结果。
- `next_prompt`：下一轮继续给模型的提示。
- `should_exit`：是否立即退出任务。

### tool_results

工具返回值会被包装成：

```python
{
  "tool_use_id": tid,
  "content": datastr
}
```

Native 模式下，`tool_use_id` 用于对齐 API 原生 tool_result。

## 一轮循环拆解

### 1. 轮次开始

每轮递增 `turn`，输出：

```text
LLM Running (Turn n) ...
```

并触发 hook：

- `turn_before`
- `llm_before`

### 2. 调用模型

```python
response_gen = client.chat(messages=messages, tools=tools_schema)
```

`client` 可能是：

- `ToolClient`
- `NativeToolClient`

主循环不关心具体 provider。

### 3. 提取工具调用

如果没有工具调用，主循环会构造特殊工具：

```python
{"tool_name": "no_tool", "args": {}}
```

这让“模型没有调用工具”也进入统一 dispatch 流程。

### 4. 分发工具

```python
handler.dispatch(tool_name, args, response)
```

真正执行在 `GenericAgentHandler.do_*` 中。

### 5. 判断下一步

- 没有 `next_prompt`：当前任务完成。
- 有 `next_prompt`：继续下一轮。
- `should_exit=True`：中断并返回。
- 达到 `max_turns`：返回 `MAX_TURNS_EXCEEDED`。

## 精读步骤

1. 先读 `StepOutcome`，理解返回值协议。
2. 读 `BaseHandler.dispatch()`，看工具名如何映射到方法。
3. 读 `agent_runner_loop()`，给每 10 行写一个注释。
4. 找到 `no_tool` 的特殊处理。
5. 找到 hook 触发点，理解插件扩展位置。
6. 找到 `messages = [{"role": "user", ...}]`，理解为什么 history 不全在这里保存。

## 动手练习

### 练习 1：画状态机

把主循环画成状态机：

```text
start
-> llm
-> tool_parse
-> dispatch
-> next_prompt?
-> continue / done / exit / max_turns
```

### 练习 2：解释 no_tool

回答：

- 为什么不直接把“无工具调用”当成最终回答？
- `do_no_tool()` 还能拦截哪些异常情况？
- 这对防止模型只输出代码块有什么帮助？

### 练习 3：改写伪代码

不用看源码，写出 `agent_runner_loop()` 的伪代码。要求不超过 40 行。

## 常见误区

### 误区 1：Agent Loop 负责所有智能

不对。Loop 只负责编排，智能主要来自：

- 模型；
- 系统提示词；
- 工具结果；
- 记忆；
- SOP。

### 误区 2：messages 每轮都包含完整历史

在这个项目中，完整 history 更多由 Session backend 管理。主循环每轮传入的新 user message 主要是下一步提示和工具结果。

## 检查清单

- [ ] 能解释 `StepOutcome` 三个字段。
- [ ] 能说出一轮 Agent Loop 的执行顺序。
- [ ] 能说明 `no_tool` 的作用。
- [ ] 能解释为什么工具分发放在 Handler。
- [ ] 能指出主循环的所有退出路径。


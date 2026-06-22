# 第 08 课：LLM 抽象层与流式响应解析

## 学习目标

- 理解 `llmcore.py` 如何适配不同 LLM API。
- 掌握 Claude SSE、OpenAI SSE、JSON 响应的解析思路。
- 理解 BaseSession、ClaudeSession、LLMSession、NativeSession 的继承关系。
- 能定位模型请求失败、流中断、工具解析失败的大致位置。

## 源码地图

- `llmcore.py::BaseSession`
- `llmcore.py::ClaudeSession`
- `llmcore.py::LLMSession`
- `llmcore.py::NativeClaudeSession`
- `llmcore.py::NativeOAISession`
- `llmcore.py::_parse_claude_sse`
- `llmcore.py::_parse_openai_sse`
- `llmcore.py::_stream_with_retry`

## 抽象层级

```text
BaseSession
├── ClaudeSession
├── LLMSession
└── NativeClaudeSession
    └── NativeOAISession

ToolClient / NativeToolClient 包装 Session，向 Agent Loop 提供统一 chat()。
```

## 核心知识点

### 1. BaseSession

BaseSession 保存通用配置：

- API Key；
- API Base；
- model；
- context window；
- history；
- timeout；
- stream；
- temperature；
- reasoning/thinking 参数。

它还提供 `ask()`，负责：

- 加锁；
- 追加 user history；
- 裁剪上下文；
- 调用 `raw_ask()`；
- 收集 content blocks。

### 2. raw_ask

不同 provider 的差异集中在 `raw_ask()`：

- Claude：`/messages`
- OpenAI：chat completions 或 responses
- NativeClaude：更接近 Claude Code 风格的工具调用
- NativeOAI：复用 OpenAI 流程

### 3. SSE 解析

流式响应不是一次性 JSON，而是很多 event。解析函数负责：

- 识别文本增量；
- 识别 thinking；
- 识别 tool_use；
- 拼接 function call arguments；
- 记录 usage；
- 检测异常中断。

### 4. 错误与重试

`_stream_with_retry()` 处理：

- HTTP 请求；
- timeout；
- API 错误；
- 重试；
- 最终错误文本返回。

错误通常会被转换成模型可见的文本，让 Agent 决定下一步。

## 精读步骤

1. 读 `BaseSession.__init__()`，记录所有配置字段。
2. 读 `BaseSession.ask()`，理解 history 追加和 raw_ask 调用。
3. 读 `_parse_claude_sse()`，标出 event 类型。
4. 读 `_parse_openai_sse()`，比较 chat_completions 和 responses。
5. 读 `_stream_with_retry()`，理解错误如何被包装。
6. 读 `_record_usage()`，确认 token 用量如何记录。

## 动手练习

### 练习 1：画响应解析流程

以 Claude SSE 为例画出：

```text
iter_lines()
-> message_start
-> content_block_start
-> content_block_delta
-> content_block_stop
-> message_stop
-> content_blocks
```

### 练习 2：分析流中断

阅读 `_parse_claude_sse()` 中 `warn` 相关逻辑，回答：

- 什么情况下会插入异常文本？
- 为什么异常文本要放在 tool_use 之前？
- 这对 Agent 后续行动有什么帮助？

### 练习 3：比较 JSON 与 SSE

列出非流式 JSON 和流式 SSE 的优缺点：

- 用户体验；
- 错误处理；
- 实现复杂度；
- 工具调用解析。

## 常见问题

### 为什么 Session 要持有 history？

因为 Agent Loop 每轮传入的 messages 并不总是完整历史。Session 需要负责 provider 侧所需的消息格式和上下文管理。

### 为什么解析函数用生成器返回？

生成器可以边收到 token 边 yield 给前端，同时在 StopIteration 时返回结构化 content blocks。

## 检查清单

- [ ] 能说清 BaseSession 的职责。
- [ ] 能解释 raw_ask 为什么由子类实现。
- [ ] 能读懂 SSE 解析的大体状态机。
- [ ] 能定位 HTTP/API 错误处理位置。
- [ ] 能解释为什么生成器既 yield 文本又 return blocks。


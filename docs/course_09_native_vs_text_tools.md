# 第 09 课：Native Tool 与文本协议 Tool

## 学习目标

- 理解 GenericAgent 同时支持原生工具调用和文本工具协议的原因。
- 掌握 `ToolClient` 与 `NativeToolClient` 的差异。
- 理解 `<tool_use>`、`<summary>`、tool_result 对齐机制。
- 能判断一个模型配置适合走 Native 还是文本协议。

## 源码地图

- `llmcore.py::ToolClient`
- `llmcore.py::NativeToolClient`
- `llmcore.py::_parse_text_tool_calls`
- `llmcore.py::openai_tools_to_claude`
- `llmcore.py::MockResponse`
- `agent_loop.py`

## 两种模式对比

| 维度 | 文本协议 Tool | Native Tool |
|---|---|---|
| 工具描述 | 写进 prompt | 传给 API tools 字段 |
| 工具调用 | 模型输出 `<tool_use>` 文本 | API 返回 tool_use block |
| 兼容性 | 高，很多模型能用 | 取决于 provider 支持 |
| 解析难度 | 需要容错解析 | 结构较稳定 |
| 历史对齐 | 文本拼接 | tool_use_id/tool_result 对齐 |

## 文本协议 Tool

`ToolClient` 会构造一个完整 prompt：

- system prompt；
- 工具调用协议；
- tools JSON；
- 历史消息；
- tool_result 文本。

模型需要按格式输出：

```xml
<tool_use>{"name": "file_read", "arguments": {"path": "README.md"}}</tool_use>
```

随后 `_parse_mixed_response()` 和 `_parse_text_tool_calls()` 把文本解析成 `MockToolCall`。

## Native Tool

`NativeToolClient` 会把工具 schema 放入 backend：

```python
self.backend.tools = tools
```

模型返回结构化 tool_use。项目再统一封装成：

- `MockToolCall`
- `MockResponse`

这样 Agent Loop 不需要区分 provider。

## summary 协议

项目要求每轮回答包含：

```xml
<summary>...</summary>
```

用途：

- 给短期历史摘要使用；
- 降低长任务中上下文丢失；
- 让每轮行动有可压缩的物理快照。

如果模型没有写 summary，`turn_end_callback()` 会自动构造一个简短摘要，并在下一轮提醒模型必须写 summary。

## 精读步骤

1. 读 `ToolClient._prepare_tool_instruction()`。
2. 读 `ToolClient._build_protocol_prompt()`。
3. 读 `ToolClient._parse_mixed_response()`。
4. 读 `NativeToolClient.chat()`。
5. 读 `NativeClaudeSession.ask()` 如何转成 `MockResponse`。
6. 对照 `agent_loop.py`，确认两种模式最终接口一致。

## 动手练习

### 练习 1：手写文本协议响应

写一个模型可能输出的文本响应，包含：

- `<thinking>`
- `<summary>`
- `<tool_use>`

然后模拟 `_parse_text_tool_calls()` 应提取出的工具名和参数。

### 练习 2：解释工具缓存

阅读 `ToolClient._prepare_tool_instruction()` 中 `last_tools` 逻辑，回答：

- 为什么重复工具描述可以省 token？
- 什么情况下要重置 `last_tools`？

### 练习 3：tool_result 对齐

阅读 `NativeToolClient.chat()` 中 `_pending_tool_ids` 逻辑，解释：

- 为什么要补空 tool_result？
- 如果少了 tool_result，严格 API 可能出现什么问题？

## 检查清单

- [ ] 能比较 Native Tool 和文本协议 Tool。
- [ ] 能解释 `<tool_use>` 文本如何变成 `MockToolCall`。
- [ ] 能说明 `MockResponse` 的统一封装价值。
- [ ] 能解释 `<summary>` 在历史压缩中的作用。
- [ ] 能判断一个模型不支持原生工具时应走哪种模式。


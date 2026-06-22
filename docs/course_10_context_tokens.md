# 第 10 课：上下文压缩、历史裁剪与 Token 控制

## 学习目标

- 理解长任务中上下文为什么会失控。
- 掌握 GenericAgent 的历史压缩和裁剪策略。
- 理解工具描述缓存、summary、working checkpoint 的配合。
- 能分析一次上下文过长问题应该从哪里入手。

## 源码地图

- `llmcore.py::compress_history_tags`
- `llmcore.py::trim_messages_history`
- `llmcore.py::_sanitize_leading_user_msg`
- `llmcore.py::_fix_messages`
- `ga.py::GenericAgentHandler._get_anchor_prompt`
- `ga.py::GenericAgentHandler.turn_end_callback`
- `ga.py::smart_format`

## 核心问题

Agent 长任务会不断积累：

- 用户输入；
- 模型思考；
- 工具调用；
- 工具结果；
- 文件内容；
- 网页内容；
- 错误堆栈；
- 计划和记忆。

如果不压缩，模型上下文会迅速超限。

## 压缩策略

### 1. 标签内容截断

`compress_history_tags()` 会压缩较早消息中的：

- `<thinking>`
- `<think>`
- `<tool_use>`
- `<tool_result>`

保留头尾，中间替换为 `[Truncated]`。

### 2. history 裁剪

`trim_messages_history()` 会：

1. 计算当前上下文字符量；
2. 先尝试压缩旧标签；
3. 仍超限则丢弃较早消息；
4. 保证消息从 user 开始；
5. 修复孤立 tool_result。

### 3. summary 历史

`turn_end_callback()` 把每轮压缩成一句：

```text
[Agent] xxx
```

`_get_anchor_prompt()` 会把最近摘要和 earlier_context 注入下一轮。

### 4. 工具描述缓存

`ToolClient` 如果发现 tools JSON 没变，会用短提示替代完整工具描述，减少重复 token。

### 5. 输出截断

`smart_format()` 会对长输出保留头尾，避免工具结果吞掉上下文。

## 精读步骤

1. 读 `compress_history_tags()`，理解正则匹配标签。
2. 读 `trim_messages_history()`，标出 cap 和 target。
3. 读 `_sanitize_leading_user_msg()`，理解孤立 tool_result 修复。
4. 读 `turn_end_callback()`，看 summary 如何进入 `history_info`。
5. 读 `_get_anchor_prompt()`，理解 working memory 注入。
6. 读 `ToolClient._prepare_tool_instruction()`，理解工具描述缓存。

## 动手练习

### 练习 1：设计压缩前后示例

写一个包含长 `<tool_result>` 的消息，手动模拟压缩后效果。

要求保留：

- 开头一小段；
- `[Truncated]`；
- 结尾一小段。

### 练习 2：分析 turn_end_callback

回答：

- 没有 summary 时它如何生成 fallback？
- 每 7 轮会追加什么提醒？
- 每 10 轮为什么要重新注入全局记忆？

### 练习 3：定位上下文膨胀来源

假设一次任务上下文突然暴涨，按优先级检查：

1. 是否读取了超大文件；
2. 是否 `web_scan` 返回过长 HTML；
3. 是否工具结果没有截断；
4. 是否模型输出了长代码块；
5. 是否工具描述缓存失效。

## 常见误区

### 误区 1：只要模型上下文足够大就不需要压缩

上下文越大，成本越高，干扰越多，检索越不稳定。压缩不是只为避免报错，也是为了提高任务稳定性。

### 误区 2：压缩就是丢信息

本项目保留“物理快照”和头尾信息，目的是把可恢复、可重读的信息留在文件或工具里，把不可恢复的关键结论写进 summary。

## 检查清单

- [ ] 能解释 `compress_history_tags()` 压缩哪些标签。
- [ ] 能说明 `trim_messages_history()` 的裁剪顺序。
- [ ] 能解释 summary 与 working memory 的区别。
- [ ] 能指出工具描述缓存如何省 token。
- [ ] 能给出上下文过长时的排查路径。


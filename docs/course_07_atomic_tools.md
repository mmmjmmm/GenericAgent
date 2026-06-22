# 第 07 课：9 个原子工具实现精读

## 学习目标

- 深入理解 GenericAgent 的 9 个核心工具。
- 掌握每个工具的输入、输出、风险和适用场景。
- 理解为什么这些工具足以组合复杂任务。
- 能从工具实现中学习最小权限和失败反馈设计。

## 源码地图

- `assets/tools_schema.json`
- `ga.py`
- `simphtml.py`
- `TMWebDriver.py`
- `assets/code_run_header.py`

## 工具列表

| 工具 | 主要能力 | 实现方法 |
|---|---|---|
| `code_run` | 执行 Python 或 shell | `do_code_run` |
| `file_read` | 读取文件片段 | `do_file_read` |
| `file_patch` | 唯一匹配替换 | `do_file_patch` |
| `file_write` | 大块写入文件 | `do_file_write` |
| `web_scan` | 获取浏览器页面简化内容 | `do_web_scan` |
| `web_execute_js` | 在真实浏览器执行 JS | `do_web_execute_js` |
| `update_working_checkpoint` | 写短期工作记忆 | `do_update_working_checkpoint` |
| `ask_user` | 请求用户介入 | `do_ask_user` |
| `start_long_term_update` | 启动长期记忆提炼 | `do_start_long_term_update` |

## 核心知识点

### 1. code_run

`code_run` 支持：

- Python 文件模式；
- PowerShell / bash 命令模式；
- 超时终止；
- stdout 流式读取；
- 输出截断；
- stop signal。

重点理解：

- Python 代码会写入临时 `.ai.py` 文件执行；
- shell 命令走系统 shell；
- 输出会被 `smart_format()` 截断；
- 长时间运行由 timeout 或 stop signal 杀掉。

### 2. file_read

`file_read` 支持：

- 起始行；
- 行数；
- keyword 附近读取；
- 行号显示；
- 文件不存在时做相似文件推荐。

这是 Agent 探索代码库的基础能力。

### 3. file_patch

`file_patch` 要求 `old_content` 在文件中唯一匹配。这样可以避免：

- 模糊替换；
- 改错位置；
- 静默破坏文件。

它失败时会要求重新 `file_read`，而不是自动猜。

### 4. file_write

`file_write` 用于大块创建或覆盖。它支持：

- `overwrite`
- `append`
- `prepend`

但精细修改应优先使用 `file_patch`。

### 5. web_scan 与 web_execute_js

这两个工具连接真实浏览器：

- `web_scan` 获取页面摘要和标签页列表；
- `web_execute_js` 执行 JS 操作页面。

项目鼓励“少全量扫描，多精确 JS”。

### 6. 记忆工具

`update_working_checkpoint` 面向短期任务上下文。

`start_long_term_update` 面向长期经验沉淀，会引导 Agent 读取记忆管理 SOP。

## 精读步骤

1. 从 `assets/tools_schema.json` 读工具描述。
2. 逐个阅读 `ga.py` 中 `do_*` 方法。
3. 对每个工具记录：
   - 参数；
   - 返回值；
   - 失败路径；
   - 安全限制。
4. 对照 `agent_loop.py` 理解工具结果如何反馈给模型。

## 动手练习

### 练习 1：工具风险分级

给 9 个工具按风险分级：

- 低风险：只读或只更新工作记忆；
- 中风险：执行代码但有超时；
- 高风险：写文件、操作网页、长期记忆更新。

解释你的分级理由。

### 练习 2：阅读 file_patch 失败策略

找出 `file_patch()` 中三类失败：

1. 文件不存在；
2. `old_content` 为空；
3. 匹配 0 次或多次。

说明每类失败为什么不应自动继续。

### 练习 3：输出截断实验

阅读 `smart_format()`，设计一个长字符串输入，预测输出会如何保留头尾。

## 检查清单

- [ ] 能列出 9 个工具及其用途。
- [ ] 能解释 `file_patch` 为什么要求唯一匹配。
- [ ] 能说明 `code_run` 如何处理超时。
- [ ] 能区分短期工作记忆和长期记忆更新。
- [ ] 能判断某个任务应该调用哪个工具。


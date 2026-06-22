# 第 11 课：分层记忆系统 L0-L4

## 学习目标

- 理解 GenericAgent 的分层记忆设计。
- 掌握 L1、L2、L3、L4 的职责边界。
- 理解记忆读取、访问统计、长期更新的触发方式。
- 能判断一条信息应该写入哪层记忆。

## 源码地图

- `memory/memory_management_sop.md`
- `memory/global_mem_insight.txt`
- `memory/global_mem.txt`
- `memory/L4_raw_sessions/`
- `ga.py::get_global_memory`
- `ga.py::log_memory_access`
- `ga.py::do_start_long_term_update`
- `assets/global_mem_insight_template*.txt`
- `assets/insight_fixed_structure*.txt`

## 分层结构

| 层级 | 名称 | 作用 |
|---|---|---|
| L0 | Meta Rules / SOP | 记忆管理规则和行为约束 |
| L1 | Insight Index | 记忆索引，帮助快速路由 |
| L2 | Global Facts | 长期稳定事实 |
| L3 | Task Skills / SOPs | 任务级可复用流程 |
| L4 | Session Archive | 原始会话和历史挖掘 |

## 核心知识点

### 1. get_global_memory

`get_global_memory()` 会把：

- 当前工作目录提示；
- memory 根目录提示；
- 固定结构说明；
- `global_mem_insight.txt`

拼入 system prompt。

这意味着 L1 是每次任务都会看到的记忆入口。

### 2. L1 与 L2/L3 的关系

L1 不应该塞满细节。它更像目录：

- 什么信息存在；
- 应该去哪个文件读；
- 何时适用。

L2/L3 才保存具体事实和 SOP。

### 3. 访问统计

`log_memory_access()` 会记录 memory 文件访问情况。这有助于后续整理常用或过期记忆。

### 4. 长期记忆更新

`start_long_term_update` 不直接写记忆，而是：

1. 读取记忆管理 SOP；
2. 让 Agent 判断哪些信息长期有效；
3. 最小化更新 L1/L2/L3。

## 精读步骤

1. 读 `memory/memory_management_sop.md`。
2. 读 `ga.py::get_global_memory()`。
3. 读 `ga.py::do_start_long_term_update()`。
4. 读 `ga.py::log_memory_access()`。
5. 打开 `memory/`，按文件名判断哪些是 L3 SOP。
6. 读 `memory/L4_raw_sessions/salient_mining_sop.md`，了解历史挖掘。

## 动手练习

### 练习 1：信息分层判断

判断下面信息应该写入哪层：

- 用户常用项目路径；
- 某网站登录按钮定位方式；
- 一次任务中临时下载文件名；
- “浏览器扩展连接失败时检查端口”；
- 最近一次会话完整日志。

### 练习 2：画记忆读取路径

画出：

```text
agentmain.get_system_prompt()
-> ga.get_global_memory()
-> assets/insight_fixed_structure*.txt
-> memory/global_mem_insight.txt
-> system prompt
```

### 练习 3：设计一条 SOP

为“排查 TUI 启动失败”设计一条 L3 SOP，要求：

- 不超过 12 步；
- 每步可验证；
- 只记录真实可复用经验，不记录临时猜测。

## 常见误区

### 误区 1：把所有内容都写入 global_mem

这会污染长期记忆，让每次任务都背负无关信息。稳定事实写 L2，任务流程写 L3，索引写 L1。

### 误区 2：任务结束就一定要更新长期记忆

只有未来能复用、经过验证、足够稳定的信息才值得更新。

## 检查清单

- [ ] 能解释 L1、L2、L3、L4 的区别。
- [ ] 能说明为什么 L1 应该短。
- [ ] 能判断一条信息应该写入哪层。
- [ ] 能解释 `start_long_term_update` 的触发目的。
- [ ] 能读懂一个 SOP 的适用场景和边界。


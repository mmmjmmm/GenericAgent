# 第 12 课：Skill / SOP 自演化机制

## 学习目标

- 理解 SOP 在 GenericAgent 中扮演的角色。
- 掌握从一次任务经验沉淀为可复用 Skill 的标准。
- 学会阅读和评估 `memory/` 中的 SOP。
- 能设计一份高质量、低噪声的任务 SOP。

## 源码地图

- `memory/*.md`
- `memory/memory_management_sop.md`
- `memory/memory_cleanup_sop.md`
- `memory/checklist_sop.md`
- `memory/review_sop.md`
- `memory/plan_sop.md`
- `memory/tmwebdriver_sop.md`
- `ga.py::do_start_long_term_update`

## SOP 的作用

SOP 是 Agent 的“经验压缩包”。它不是普通文档，而是用于未来任务执行的操作规程。

高质量 SOP 应该具备：

- 触发条件清晰；
- 步骤短而可执行；
- 每步有验证方式；
- 包含关键坑点；
- 不包含泛泛常识；
- 不记录一次性临时变量。

## 自演化流程

```text
完成任务
-> 判断是否有长期可复用经验
-> start_long_term_update
-> 读取 memory_management_sop
-> 判断写 L2 还是 L3
-> 最小化 patch 记忆文件
-> 同步更新 L1 索引
```

## 核心知识点

### 1. SOP 不是越长越好

GenericAgent 的设计强调 token 成本。SOP 应该记录“以后不想再踩一次的坑”，而不是把完整任务日志搬进去。

### 2. SOP 应该可执行

坏写法：

```text
注意检查环境。
```

好写法：

```text
先运行 `python --version`，若为 3.14，切换到 3.11/3.12 后再安装 UI 依赖。
```

### 3. SOP 要有边界

每份 SOP 都应说明：

- 什么时候用；
- 什么时候不用；
- 依赖哪些文件或工具；
- 失败时如何降级。

### 4. 清理和压缩

`memory/memory_cleanup_sop.md` 用于避免记忆膨胀。核心原则是存在性编码：索引指向细节，不在索引里塞细节。

## 精读步骤

1. 读 `memory/memory_management_sop.md`，理解记忆写入标准。
2. 读 `memory/memory_cleanup_sop.md`，理解压缩原则。
3. 任选三份 SOP：
   - `tmwebdriver_sop.md`
   - `review_sop.md`
   - `plan_sop.md`
4. 对每份 SOP 标注：
   - 触发条件；
   - 核心步骤；
   - 风险提示；
   - 完成标准。

## 动手练习

### 练习 1：SOP 质量评审

选择一份 `memory/*.md`，回答：

- 它解决什么任务？
- 哪些步骤最关键？
- 是否有过期或模糊内容？
- 是否可以压缩？

### 练习 2：从任务日志提炼 SOP

假设任务日志包含 20 步，其中 15 步是探索，5 步是最终有效路径。

写出 SOP 时只保留：

- 前置条件；
- 最终有效路径；
- 失败检查点；
- 不要保留探索岔路。

### 练习 3：设计触发条件

为一个“飞书文档读取失败排查 SOP”写触发条件：

- 用户给出飞书文档 URL；
- 读取失败；
- 出现权限或 token 错误；
- 需要判断用 user 还是 bot 身份。

## 检查清单

- [ ] 能解释 SOP 与普通文档的区别。
- [ ] 能判断任务经验是否值得长期保存。
- [ ] 能写出可执行、短小、有边界的 SOP。
- [ ] 能说明 L1 索引与 L3 SOP 如何同步。
- [ ] 能识别记忆膨胀和重复记录问题。


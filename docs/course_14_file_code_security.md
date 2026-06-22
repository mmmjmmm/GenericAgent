# 第 14 课：文件、代码执行与安全边界

## 学习目标

- 理解 Agent 文件读写和代码执行的安全风险。
- 掌握 `file_read`、`file_patch`、`file_write`、`code_run` 的边界。
- 理解项目如何通过唯一匹配、超时、截断、用户询问降低风险。
- 能为新增工具设计安全约束。

## 源码地图

- `ga.py::code_run`
- `ga.py::file_read`
- `ga.py::file_patch`
- `ga.py::do_file_write`
- `ga.py::expand_file_refs`
- `assets/code_run_header.py`
- `CONTRIBUTING.md`
- `memory/code_review_principles.md`

## 核心风险

Agent 拥有本地工具能力后，主要风险包括：

- 误改文件；
- 覆盖用户改动；
- 执行危险命令；
- 输出泄露敏感信息；
- 长进程卡死；
- 大文件撑爆上下文；
- 自动化操作不可逆。

## 工具边界设计

### 1. file_read 是探索入口

任何修改前都应该先读文件。`file_read` 提供：

- 行号；
- 分段读取；
- keyword 搜索；
- 截断提示；
- 相似文件推荐。

### 2. file_patch 用唯一匹配保护修改

`file_patch` 不接受模糊替换。它要求：

- `old_content` 非空；
- 文件中匹配次数为 1；
- 否则返回错误。

这是防止“看起来改了，其实改错位置”的关键。

### 3. file_write 用于大块写入

`file_write` 更危险，因为它可以覆盖整个文件。适用场景：

- 新建文件；
- 生成完整文档；
- 大规模重写。

不适合精细修改已有代码。

### 4. code_run 有超时和输出截断

`code_run` 会：

- 用临时文件运行 Python；
- shell 命令走 bash / PowerShell；
- 读取 stdout；
- 超时 kill；
- 根据 `maxlen` 截断输出。

### 5. ask_user 是安全出口

遇到不可判断、不可逆、需要权限或存在多种解释的情况，应调用 `ask_user`。

## 精读步骤

1. 读 `file_read()`，理解读取和推荐逻辑。
2. 读 `file_patch()`，找出全部失败条件。
3. 读 `do_file_write()`，理解内容提取和写入模式。
4. 读 `expand_file_refs()`，理解 `{{file:path:start:end}}`。
5. 读 `code_run()`，理解临时文件、进程、超时、输出读取。
6. 读 `CONTRIBUTING.md`，理解项目对简洁和小变更半径的要求。

## 动手练习

### 练习 1：设计安全修改流程

给定任务“修改 `agent_loop.py` 的某个提示文本”，写出安全流程：

1. `file_read` 定位；
2. 复制足够上下文；
3. `file_patch` 唯一替换；
4. 再次读取验证；
5. 运行最小检查。

### 练习 2：分析覆盖风险

列出 `file_write overwrite` 不应该用于哪些场景：

- 用户正在编辑的源码；
- 只需改一行的配置；
- 未完整读取过的文件；
- 存在未确认格式的文件。

### 练习 3：code_run 威胁建模

分析下面命令风险：

```bash
rm -rf temp/*
git reset --hard
python script.py
curl https://example.com/install.sh | sh
```

分别说明是否需要用户确认、是否可替代、如何降低风险。

## 检查清单

- [ ] 能解释为什么修改前必须读文件。
- [ ] 能说明 `file_patch` 的唯一匹配保护。
- [ ] 能区分 `file_patch` 和 `file_write` 的适用场景。
- [ ] 能解释 `code_run` 的超时和输出截断。
- [ ] 能为危险操作设计用户确认点。


# 第 15 课：自主任务、反射任务与调度器

## 学习目标

- 理解 `reflect/` 目录的用途。
- 掌握自主运行、目标模式、定时任务的基本机制。
- 理解反射脚本如何通过 `check()` 触发新任务。
- 能设计一个简单的周期性检查任务。

## 源码地图

- `reflect/autonomous.py`
- `reflect/scheduler.py`
- `reflect/goal_mode.py`
- `reflect/checklist_master.py`
- `reflect/agent_team_worker.py`
- `memory/autonomous_operation_sop.md`
- `memory/scheduled_task_sop.md`
- `memory/goal_mode_sop.md`
- `frontends/tui_v3.py`

## 核心概念

### 1. 反射任务

反射任务不是用户直接输入，而是后台脚本周期性检查状态，在条件满足时给 Agent 发任务。

典型接口是：

```python
def check():
    ...
```

如果 `check()` 返回任务内容，Agent 就可以执行。

### 2. 自主模式

`reflect/autonomous.py` 很短，真正规则在：

- `memory/autonomous_operation_sop.md`
- `memory/autonomous_operation_sop/task_planning.md`

这体现了项目哲学：机制留在代码，策略写在记忆。

### 3. 定时任务

`reflect/scheduler.py` 负责：

- 解析任务配置；
- 判断 cooldown；
- 检查上次运行；
- 到点触发。

### 4. Goal Mode

`reflect/goal_mode.py` 维护目标状态，配合 `memory/goal_mode_sop.md` 执行长期目标。

它关注：

- 目标；
- 预算；
- 状态；
- 完成条件；
- on_done 回调。

## 精读步骤

1. 读 `reflect/autonomous.py`，感受代码有多薄。
2. 读 `memory/autonomous_operation_sop.md`，理解策略主体。
3. 读 `reflect/scheduler.py`，标出 `_parse_cooldown()` 和 `check()`。
4. 读 `memory/scheduled_task_sop.md`。
5. 读 `reflect/goal_mode.py`，理解状态文件读写。
6. 搜索 `frontends/tui_v3.py` 中 `/autorun`、`/scheduler`、`/goal`。

## 动手练习

### 练习 1：写一个反射脚本草稿

设计一个 `check()`：

- 每小时检查某个文件是否存在；
- 如果存在，返回“读取并总结这个文件”；
- 如果不存在，返回 None。

重点是接口设计，不要求接入运行。

### 练习 2：分析调度 cooldown

阅读 `_parse_cooldown(repeat)`，说明它如何把 repeat 配置转换为间隔。

### 练习 3：目标状态建模

为“持续整理 docs 中的学习笔记”设计 goal state：

- goal；
- budget；
- done condition；
- current status；
- last action。

## 常见误区

### 误区 1：自主模式等于无限自动执行

不是。SOP 中有权限边界和等待用户审查的要求。自主能力必须有预算、目标和停止条件。

### 误区 2：反射脚本应该写复杂业务逻辑

反射脚本只负责触发条件。复杂策略应该交给 Agent 和 SOP。

## 检查清单

- [ ] 能解释 reflect 脚本的职责。
- [ ] 能说明 `check()` 的触发模型。
- [ ] 能读懂 scheduler 的 cooldown 判断。
- [ ] 能区分自主模式和目标模式。
- [ ] 能设计一个有停止条件的后台任务。


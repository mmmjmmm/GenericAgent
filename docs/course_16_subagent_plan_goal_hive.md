# 第 16 课：Subagent、Plan、Goal 与 Hive 模式

## 学习目标

- 理解 GenericAgent 如何组织复杂任务。
- 掌握 Subagent 文件 IO 协议。
- 理解 Plan Mode 的探索、规划、执行、验证闭环。
- 区分 Goal Mode 和 Hive Mode 的适用场景。

## 源码地图

- `memory/subagent.md`
- `memory/plan_sop.md`
- `memory/goal_mode_sop.md`
- `memory/goal_hive_sop.md`
- `memory/goal_hive_master_duty.md`
- `frontends/conductor.py`
- `frontends/conductor.html`
- `frontends/plan_state.py`
- `reflect/goal_mode.py`
- `reflect/agent_team_worker.py`

## 模式总览

| 模式 | 适用场景 | 核心机制 |
|---|---|---|
| Subagent | 独立探索、验证、并行处理 | 文件 IO 协议 |
| Plan Mode | 多步骤开发或复杂任务 | plan.md + 检查点 |
| Goal Mode | 长期目标持续推进 | goal_state.json |
| Hive Mode | 多 worker 协作长期任务 | master + workers |
| Conductor | 前端可视化多 subagent 编排 | HTTP/HTML 控制台 |

## Subagent

`memory/subagent.md` 定义了通过文件交换任务和结果的协议。

典型用途：

- 独立验证；
- 并行搜索；
- 不污染主上下文的探索；
- 对某个假设做反证。

关键思想：主 Agent 不把所有细节塞进自己上下文，而是把可独立完成的工作交给子任务。

## Plan Mode

`memory/plan_sop.md` 把复杂任务拆成：

1. 探索态；
2. 规划态；
3. 执行态；
4. 验证态；
5. 失败处理。

核心产物是 `plan.md`，其中待办项用 `[ ]` 和 `[x]` 表示。

`ga.py::do_no_tool()` 会在 Plan Mode 中拦截过早完成声明，要求先执行验证。

## Goal Mode

Goal Mode 面向长期目标。它不是一次对话完成，而是维护状态并周期性推进。

常见字段：

- 目标；
- 预算；
- 当前阶段；
- 已完成内容；
- 下一步；
- 停止条件。

## Hive Mode

Hive Mode 是更强的多 worker 模式。它要求 Master：

- 探测；
- 设计；
- 执行；
- 检查；
- 分配 worker；
- 合并结果；
- 控制失稳风险。

## 精读步骤

1. 读 `memory/subagent.md`，理解文件 IO 协议。
2. 读 `memory/plan_sop.md`，画出五阶段流程。
3. 读 `frontends/plan_state.py`，理解如何解析 plan 状态。
4. 读 `reflect/goal_mode.py`，理解 goal state。
5. 读 `memory/goal_hive_sop.md` 和 `goal_hive_master_duty.md`。
6. 读 `frontends/conductor.py`，了解多 agent 编排入口。

## 动手练习

### 练习 1：拆分任务

把“完整学习 GenericAgent 并写课程文档”拆成：

- 主任务；
- 可派给 subagent 的探索任务；
- 需要主 Agent 决策的任务；
- 验证任务。

### 练习 2：写 plan.md 模板

写一个包含以下章节的 plan：

- 探索发现；
- 执行计划；
- 验证检查点；
- 风险和回滚；
- 待办清单。

### 练习 3：设计 Hive 角色

为一个“持续维护项目文档”的 Hive 设计：

- Master 职责；
- Worker A：源码变更扫描；
- Worker B：文档一致性检查；
- Worker C：示例运行验证。

## 常见误区

### 误区 1：所有任务都需要 Plan Mode

简单任务不需要。Plan Mode 适合跨文件、多步骤、高风险、需要验证闭环的任务。

### 误区 2：Subagent 是为了更多输出

Subagent 的价值是隔离上下文和并行验证，不是制造更多文本。

## 检查清单

- [ ] 能解释 Subagent 文件 IO 协议。
- [ ] 能画出 Plan Mode 五阶段。
- [ ] 能说明 Plan Mode 为什么要求验证。
- [ ] 能区分 Goal Mode 和 Hive Mode。
- [ ] 能判断一个任务是否值得拆给 Subagent。


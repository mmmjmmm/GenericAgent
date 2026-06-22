# 第 03 课：API Key、多模型配置与会话选择

## 学习目标

- 理解 `mykey.py` / `mykey.json` 的加载机制。
- 掌握不同模型配置命名如何影响 Session 类型。
- 理解 MixinSession 的多模型 fallback 机制。
- 能解释 `/llm` 或 `next_llm()` 背后的模型切换过程。

## 源码地图

- `mykey_template.py`：中文配置模板。
- `mykey_template_en.py`：英文配置模板。
- `llmcore.py`：配置加载、Session 构造、多模型 fallback。
- `agentmain.py`：`load_llm_sessions()`、`next_llm()`、`list_llms()`。
- `docs/GETTING_STARTED.md`：配置示例说明。

## 核心知识点

### 1. 配置加载顺序

`llmcore._load_mykeys()` 会优先尝试：

1. 导入 `mykey.py`；
2. 如果没有，再读取 `mykey.json`；
3. 如果 JSON 中有 `remote_url`，则远程拉取配置。

这让配置可以本地写死，也可以统一托管。

### 2. 配置命名驱动 Session 类型

`llmcore.resolve_session()` 通过配置名判断使用哪个 Session：

- 名字包含 `native` 和 `claude`：`NativeClaudeSession`
- 名字包含 `native` 和 `oai`：`NativeOAISession`
- 名字包含 `claude`：`ClaudeSession`
- 名字包含 `oai`：`LLMSession`

这是一种轻量约定：配置名本身携带路由信息。

### 3. Native 与非 Native

非 Native 模式通常把工具协议写进 prompt，让模型输出 `<tool_use>` 文本块。

Native 模式则尽量走模型 API 原生工具调用字段，再把响应统一包装成 `MockResponse`。

### 4. MixinSession

MixinSession 支持多个模型配置按顺序 fallback：

- 主模型优先；
- 失败后切到备用模型；
- 一段时间后 spring back 到主模型；
- 要求混合的 Session 同属 Native 或非 Native。

## 精读步骤

1. 读 `mykey_template.py`，记录每类配置字段。
2. 读 `llmcore._load_mykeys()` 和 `reload_mykeys()`。
3. 读 `llmcore.resolve_session()` 和 `resolve_client()`。
4. 读 `agentmain.GenericAgent.load_llm_sessions()`。
5. 读 `agentmain.GenericAgent.next_llm()`，理解切模型时如何继承 history。
6. 读 `MixinSession`，理解 fallback 不是简单轮询。

## 动手练习

### 练习 1：画配置到客户端的映射

写出下面流程：

```text
mykey.py
-> reload_mykeys()
-> resolve_client()
-> BaseSession 子类
-> ToolClient 或 NativeToolClient
-> GenericAgent.llmclients
```

### 练习 2：解释配置名

假设有这些配置名：

```python
native_claude_api = {...}
native_oai_api = {...}
claude_api = {...}
oai_api = {...}
mixin_config = {...}
```

分别说明会进入哪条构造路径。

### 练习 3：模拟模型切换

阅读 `next_llm()` 后回答：

- 为什么要复制旧 history？
- 为什么某些模型会加载中文工具 schema？
- `llm_no` 越界时如何处理？

## 常见问题

### 为什么不把 provider 写成显式字段？

当前项目用配置名约定来保持代码短小。优点是简单，缺点是配置名必须遵守规则。

### 为什么 MixinSession 不能混用 Native 和非 Native？

因为两类会话的工具调用协议不同。混用会导致工具调用、tool_result 对齐和 history 格式不一致。

## 检查清单

- [ ] 能说清 `mykey.py` 与 `mykey.json` 的加载顺序。
- [ ] 能根据配置名判断 Session 类型。
- [ ] 能解释 `ToolClient` 和 `NativeToolClient` 的区别。
- [ ] 能说明 MixinSession 的 fallback 策略。
- [ ] 能定位模型配置错误通常发生在哪些函数。


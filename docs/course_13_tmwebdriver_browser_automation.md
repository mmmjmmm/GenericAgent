# 第 13 课：TMWebDriver 与真实浏览器自动化

## 学习目标

- 理解 TMWebDriver 如何连接真实浏览器标签页。
- 掌握 `web_scan` 与 `web_execute_js` 的职责分工。
- 理解 `simphtml` 如何压缩网页内容。
- 能排查浏览器连接失败、标签页切换、JS 执行失败等问题。

## 源码地图

- `TMWebDriver.py`
- `simphtml.py`
- `ga.py::web_scan`
- `ga.py::web_execute_js`
- `assets/tmwd_cdp_bridge/`
- `memory/tmwebdriver_sop.md`
- `memory/web_setup_sop.md`
- `memory/vue3_component_sop.md`

## 架构概览

```text
GenericAgent
-> ga.web_scan / web_execute_js
-> TMWebDriver
-> WebSocket / HTTP long-poll / extension bridge
-> real browser tab
-> JS execution result
```

## 核心知识点

### 1. 真实浏览器

TMWebDriver 控制的是用户真实浏览器标签页，而不是新启动的无状态浏览器。这带来：

- 可复用登录态；
- 更接近真实用户环境；
- 不需要下载浏览器二进制；
- 但需要安装或连接浏览器端桥接脚本。

### 2. Session 管理

`TMWebDriver.Session` 保存：

- session id；
- url；
- title；
- 连接类型；
- WebSocket client 或 HTTP queue；
- disconnect 状态。

`TMWebDriver` 维护多个 session，并有默认活动标签页。

### 3. web_scan

`web_scan` 用于观察：

- 标签页列表；
- 当前页面简化 HTML；
- text_only 页面文本。

它会调用 `simphtml.get_html()` 来减少 token。

### 4. web_execute_js

`web_execute_js` 用于行动：

- 点击；
- 输入；
- 读取 DOM；
- 操作 Vue/React 组件；
- 触发下载；
- 获取页面状态。

项目建议：能用 JS 精确读取和操作，就不要频繁全量 scan。

### 5. simphtml

`simphtml.py` 负责：

- 清理隐藏元素；
- 提取主内容；
- 压缩 HTML；
- 监控 JS 执行前后页面变化；
- 返回更适合模型阅读的页面摘要。

## 精读步骤

1. 读 `TMWebDriver.Session`。
2. 读 `TMWebDriver.__init__()`，理解本地与 remote 模式。
3. 读 `start_ws_server()` 和 `start_http_server()`。
4. 读 `execute_js()`，理解消息发送、ack、result 等待。
5. 读 `ga.py::web_scan` 与 `ga.py::web_execute_js`。
6. 读 `simphtml.get_html()` 与 `execute_js_rich()`。
7. 读 `memory/tmwebdriver_sop.md`，记录常见排查项。

## 动手练习

### 练习 1：画连接流程

画出浏览器端连接到 Python 端的流程：

```text
browser tab
-> ready/ext_ready/tabs_update
-> TMWebDriver.sessions
-> default_session_id
-> execute_js
-> result
```

### 练习 2：区分观察和行动

判断以下任务用 `web_scan` 还是 `web_execute_js`：

- 查看当前有哪些标签页；
- 点击页面按钮；
- 获取页面正文；
- 读取某个 input 的 value；
- 切换到指定 tab；
- 上传文件。

### 练习 3：排查连接失败

按顺序检查：

1. Python 端是否启动 `127.0.0.1:18765`；
2. 浏览器扩展是否安装；
3. 当前是否有活动标签页；
4. session 是否断开；
5. 是否被浏览器安全策略或页面 CSP 限制；
6. 是否需要 CDP bridge。

## 常见误区

### 误区 1：web_scan 看到的就是完整页面

不是。`simphtml` 会过滤隐藏、浮层、边栏等内容。需要精确元素时，应使用 JS 查询 DOM。

### 误区 2：JS click 等于真实用户点击

有些网站检查 `isTrusted` 或完整输入事件生命周期。遇到这类情况要读 `tmwebdriver_sop.md` 中的 CDP 点击方案。

## 检查清单

- [ ] 能解释 TMWebDriver 与 Playwright 的差异。
- [ ] 能说明 Session 如何管理多个标签页。
- [ ] 能区分 `web_scan` 和 `web_execute_js`。
- [ ] 能解释 `simphtml` 为什么要压缩 HTML。
- [ ] 能列出浏览器连接失败的排查路径。


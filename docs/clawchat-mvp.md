# ClawChat MVP 方案

## 定位

`ClawChat` 暂定为 `zeroclaw` 的配套客户端，而不是新的 agent runtime。

核心原则：

- `zeroclaw` 继续运行在服务器端
- AI provider key 只保留在服务器端
- 客户端只做配对、聊天、状态查看和常用管理动作
- 不再把 Telegram、Discord、Slack 之类渠道作为主入口

这意味着 `ClawChat` 实际上是：

- 一个面向人类的控制端
- 一个基于现有 `zeroclaw gateway` 的轻量 companion app
- 一个“扫二维码或输入配对码即可接入服务器”的产品层

## 命名判断

`ClawChat` 可以先用，优点是：

- 直观
- 容易理解成“和 Claw 对话的客户端”
- 适合 MVP 阶段

当前需要保留的判断：

- 如果后面产品不止聊天，还要覆盖服务管理、审批、巡检、节点列表，`ClawChat` 会略偏“聊天工具”语义
- 所以现阶段建议：
- 产品代号继续用 `ClawChat`
- 后续如果功能明显超出聊天，再决定是否升级为 `Claw` / `Claw Control` / `Claw Companion`

## 产品目标

MVP 只解决一个问题：

让用户不需要配置各种机器人和各种渠道，只需要把一台服务器配对进来，就能直接使用 `zeroclaw`。

MVP 不做的事：

- 不重新实现 agent loop
- 不在客户端存 AI provider key
- 不在第一阶段接入第三方 IM 平台
- 不在第一阶段开放高风险全权限运维动作

## 用户路径

最小用户路径应该是：

1. 打开 `ClawChat`
2. 添加服务器
3. 输入地址或扫描二维码
4. 输入一次性配对码
5. 获取设备 token
6. 直接进入聊天和管理界面

对于用户来说，最终只保留两个概念：

- 服务器地址
- 配对码

## 技术边界

### 服务端

沿用现有 `zeroclaw gateway` 能力：

- `GET /health`
- `POST /pair`
- `GET /api/*`
- `GET /ws/chat`
- 设备管理 API

服务端继续负责：

- AI provider 配置
- 模型选择
- 工具调用
- 审计
- 会话和任务执行

### 客户端

`ClawChat` 负责：

- 配对
- token 保存
- 聊天 UI
- 服务器列表
- 常用状态卡片
- 常用操作入口

客户端不负责：

- 直接调用模型
- 保存 provider key
- 重写 `zeroclaw` 的调度、记忆、工具系统

## 推荐实现顺序

### 第一阶段：PWA

先做 Web/PWA，而不是直接做原生 App。

原因：

- 复用 `zeroclaw` 现有 web/gateway 能力最快
- 可以先跑通配对和聊天协议
- 适合快速迭代 UI 和权限模型
- 以后再打包成移动端成本更低

建议技术栈：

- React
- TypeScript
- PWA
- WebSocket
- TanStack Query 或等价请求层

### 第二阶段：原生壳

如果 PWA 跑顺，再考虑：

- Capacitor
- React Native
- Tauri Mobile

原则不是“先原生”，而是“先把协议和产品流程跑通”。

## MVP 页面

MVP 建议只做 5 个页面：

### 1. 配对页

能力：

- 输入服务器地址
- 输入配对码
- 二维码扫码
- 首次配对完成后保存设备 token

### 2. 会话页

能力：

- 连接 `/ws/chat`
- 多轮会话
- 显示工具执行中状态
- 显示错误和重连状态

### 3. 概览页

能力：

- 服务器在线状态
- 最近心跳
- 当前模型
- 最近任务
- 最近告警

### 4. 操作页

能力：

- 预设操作入口
- 只放高频、低风险动作
- 例如：
- 查看 launchd 状态
- 查看 Docker 状态
- 查看磁盘空间
- 查看最近异常

### 5. 设备页

能力：

- 当前设备列表
- 上次在线时间
- 吊销设备
- 旋转 token

## API 对接建议

优先复用现有接口，不自己发明一套新的：

- 配对：`POST /pair`
- 健康检查：`GET /health`
- 状态：`GET /api/status`
- 健康：`GET /api/health`
- 设备列表：`GET /api/devices`
- 吊销设备：`DELETE /api/devices/{id}`
- Chat：`GET /ws/chat`

MVP 不建议立即新增服务端协议，除非现有接口不够。

## 安全模型

这是 `ClawChat` 最重要的一层。

必须坚持：

- provider key 只在服务器
- 每个设备单独 token
- token 可以单独吊销
- token 存系统安全存储
- 高风险操作要二次确认

建议默认策略：

- 默认只开放读操作和预设动作
- 写操作按白名单逐步开放
- 后续增加“需要确认”的动作类别
- 远程接入优先走 Tailscale 或你自己的隧道，不建议直接裸露公网

## 二维码配对建议

二维码内容不要塞入敏感 token，只放：

- 服务器地址
- 一次性配对码
- 可选设备名提示

建议结构：

```json
{
  "type": "clawchat_pair",
  "server": "https://your-gateway.example.com",
  "code": "ABCDEFGH"
}
```

扫码后由客户端调用 `POST /pair`，换回设备 token。

## 第一版里程碑

### Milestone 1

- 写清协议和页面结构
- 跑通手动输入地址 + 配对码
- 跑通 `/ws/chat`

### Milestone 2

- 增加二维码扫码
- 增加服务器列表
- 增加状态页

### Milestone 3

- 增加设备管理
- 增加常用运维预设动作
- 增加移动端适配

## 当前结论

`ClawChat` 不该做成新的“平台适配器集合”，而应该做成：

- `zeroclaw` 的官方客户端
- 以配对为入口
- 以 WebSocket 会话为主交互
- 以服务器为中心，而不是以第三方机器人为中心

这条路开发成本更低，也更符合你现在“只想直接管理自己的服务器”的目标。

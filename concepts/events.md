# 事件系统

UniQQ SDK 基于事件驱动架构，插件通过订阅事件来响应各种机器人活动。

## 事件总线

`Context.Events` 是事件总线实例，用于**订阅**和**取消订阅**事件。

```csharp
// 订阅事件（在 OnEnable 中）
Context.Events.On<GroupMessageEvent>(OnGroupMessage);

// 取消订阅（在 OnDisable 中）
Context.Events.Off<GroupMessageEvent>();
```

### 事件订阅规则

- 所有事件订阅在 **OnEnable** 中统一注册
- 事件处理方法使用 `async Task`
- !!必须在 **OnDisable** 中取消订阅，否则会造成内存泄漏和事件误发！
- !!禁止同步阻塞事件处理

---

## 事件分类

事件分为四大类：

| 分类 | 命名空间 | 说明 |
|------|----------|------|
| Meta | `Events.Meta` | 机器人元事件（上线、下线、心跳） |
| Message | `Events.MessageEvents` | 消息事件（群聊、私聊、临时会话） |
| Notice | `Events.Notice` | 通知事件（加群、退群、禁言等） |
| Request | `Events.Request` | 请求事件（好友申请、加群申请等） |
| System | `Events.System` | 系统事件（插件加载/卸载） |

---

## Meta 事件（元事件）

```csharp
private async Task OnBotOnline(BotOnlineEvent e)
{
    // e.BotUin     - 机器人 QQ 号
    // e.Timestamp  - 上线时间戳
    await Context.WriteLog($"机器人 {e.BotUin} 已上线");
}
```

### BotOnlineEvent — 机器人上线

**触发时机**：机器人账号登录成功

| 属性 | 类型 | 说明 |
|------|------|------|
| `BotUin` | long | 机器人 QQ 号 |
| `Timestamp` | long | 下线时间戳 |

### BotOfflineEvent — 机器人下线

**触发时机**：机器人账号断开连接

| 属性 | 类型 | 说明 |
|------|------|------|
| `BotUin` | long | 机器人 QQ 号 |
| `Timestamp` | long | 下线时间戳 |
| `Message` | string | 下线原因描述 |

### HeartbeatEvent — 心跳事件

**触发时机**：定时心跳包（可用于监控机器人状态）

| 属性 | 类型 | 说明 |
|------|------|------|
| `BotUin` | long | 机器人 QQ 号 |
| `Timestamp` | long | 心跳时间 |
| `Online` | boolean | 当前是否在线 |
| `Latency` | long | 延迟（毫秒） |

---
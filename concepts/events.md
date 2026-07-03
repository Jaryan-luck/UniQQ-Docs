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
```csharp
UniQQ.SDK.Events.Meta //机器人元事件（上线、下线、心跳）
UniQQ.SDK.Events.Message //消息事件（群聊、私聊、临时会话）
UniQQ.SDK.Events.Notice //通知事件（加群、退群、禁言等）
UniQQ.SDK.Events.System //系统事件（插件加载/卸载）
```
---

## Meta 事件（元事件）

### BotOnlineEvent — 机器人上线

**触发时机**：机器人账号登录成功

| 属性 | 类型 | 说明 |
|------|------|------|
| `BotUin` | long | 机器人 QQ 号 |
| `Timestamp` | long | 下线时间戳 |

> **示例-捕获上线**：
```csharp
private async Task OnBotOnline(BotOnlineEvent e)
{
    // e.BotUin     - 机器人 QQ 号
    // e.Timestamp  - 上线时间戳
    await Context.WriteLog($"机器人 {e.BotUin} 已上线");
}
```

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

## Message 事件（消息事件）

### GroupMessageEvent — 群消息事件 ⭐

最常用的事件，接收所有群聊消息。

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 群号 |
| `User_Id` | long | 发送者 QQ |
| `User_NickName` | string | 发送者昵称 |
| `User_Card` | string | 群名片 |
| `User_Role` | MemberRole | 角色（Member/Admin/Owner） |
| `Message` | Message | 消息内容 |
| `MessageId` | long | 消息 ID |
| `IsAtBot` | boolean | 是否 @ 了机器人 |

> **示例-回复消息**：
```csharp
private async Task OnGroupMessage(GroupMessageEvent e)
{
    // 方式一：通过 Context 发送
    await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id,
        MessageBuilder.Text("回复消息"));
    
    // 方式二：使用 ReplyAsync（便捷回复）Reference：是否引用触发的消息
    await e.ReplyAsync(MessageBuilder.Text("快捷回复"), Reference: true);
}
```

### PrivateMessageEvent — 私聊消息事件

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `User_Id` | long | 发送者 QQ |
| `User_NickName` | string | 发送者昵称 |
| `User_Sex` | string | 性别 |
| `User_Age` | int | 年龄 |
| `Message` | Message | 消息内容 |
| `MessageId` | long | 消息 ID |

### TempMessageEvent — 临时会话消息事件

群内临时会话（非好友之间的私聊）。

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 发送者 QQ |
| `User_NickName` | string | 发送者昵称 |
| `User_Sex` | string | 性别 |
| `User_Age` | int | 年龄 |
| `Message` | Message | 消息内容 |
| `MessageId` | long | 消息 ID |


---

## Notice 事件（通知事件）

### 群成员变动
#### MemberJoinEvent - 成员入群
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 加入者 QQ |
| `Operator_Id` | string | 操作者 QQ |

#### MemberLeaveEvent - 成员退群
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 退群者 QQ |

#### MemberKickEvent - 成员被踢
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 被踢成员 QQ |
| `Operator_Id` | string | 操作者 QQ（踢人者） |

#### BotKickEvent - 机器人被踢
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 被踢的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `Operator_Id` | string | 操作者 QQ（踢出机器人者） |

#### BotLeaveGroupEvent - 机器人主动退群
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 主动退群的机器人 QQ |
| `Group_Id` | long | 退出的群号 |

#### BotJoinGroupEvent - 机器人入群
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 入群的机器人 QQ |
| `Group_Id` | long | 加入的群号 |
| `Operator_Id` | string | 操作者 QQ（邀请者或管理员） |
| `IsInvited` | boolean | 是否通过邀请入群 |


### 群管理操作
#### MemberMuteEvent - 成员被禁言
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 被禁言的成员 QQ |
| `Duration` | long | 禁言时长（秒） |
| `Operator_Id` | string | 操作者 QQ（执行禁言的管理员或群主） |

#### MemberUnmuteEvent - 成员解除禁言
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 被解除禁言的成员 QQ |
| `Operator_Id` | string | 操作者 QQ（执行解除的管理员或群主） |

#### MemberSetAdminEvent - 成员被设为管理
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 被设为管理的成员 QQ |
| `Operator_Id` | string | 操作者 QQ（执行设置的管理员或群主） |

#### MemberUnsetAdminEvent - 成员取消管理
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 被取消管理的成员 QQ |
| `Operator_Id` | string | 操作者 QQ（执行取消的管理员或群主） |


> **示例 - 禁言通知**：
```csharp
private async Task OnMemberMute(MemberMuteEvent e)
{
    if (e.User_Id != 0) // 排除全员禁言
    {
        await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id,
            MessageBuilder.Text($"{e.User_Id} 被禁言了 {e.Duration} 秒"));
    }
}
```
### 消息相关
#### GroupMessageRecallEvent - 群消息被撤回
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `MessageId` | long | 被撤回消息的 ID |
| `Operator_Id` | string | 撤回操作者 QQ（群主、管理员或消息发送者本人） |
| `User_Id` | long | 原消息发送者 QQ |

#### FriendMessageRecallEvent - 好友消息被撤回
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `MessageId` | long | 被撤回消息的 ID |
| `User_Id` | long | 原消息发送者 QQ（即好友） |

#### EssenceMessageEvent - 消息精华状态变更
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `MessageId` | long | 精华操作涉及的消息 ID |
| `Operator_Id` | string | 执行操作的管理员 QQ |
| `IsAdd` | boolean | `true` 表示设为精华，`false` 表示取消精华 |
| `User_Id` | long | 原消息发送者 QQ |

#### GroupPokeEvent - 群内戳一戳
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `User_Id` | long | 发起戳一戳的成员 QQ |
| `Target_Id` | long | 被戳的成员 QQ |

#### FriendPokeEvent - 好友戳一戳
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | NULL |
| `User_Id` | long | 发起戳一戳的好友 QQ |
| `Target_Id` | long | 被戳的好友 QQ |

### 群变动
#### GroupNameChangedEvent - 群名称变更
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 群号 |
| `NewName` | string | 变更后的新群名称 |
| `Operator_Id` | string | 执行操作的管理员 QQ |

#### ReactionEvent - 消息回应（表情）
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 来源群号 |
| `MessageId` | long | 被回应消息的 ID |
| `ReactionId` | string | 回应表情的标识 ID |
| `IsAdd` | boolean | `true` 表示添加回应，`false` 表示移除回应 |
| `Operator_Id` | string | 操作者 QQ |

### 文件相关
#### FileUploadEvent - 群文件上传
| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 文件上传的目标群号 |
| `FileId` | string | 文件的唯一标识 ID（可用于下载或引用） |
| `FileName` | string | 上传的文件名（含扩展名） |
| `FileSize` | string | 上传的文件大小 |
| `User_Id` | long | 文件发送者 QQ |

## 好友相关

### FriendAddedEvent — 新增好友

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `User_Id` | long | 新增好友 QQ |

---

## Request 事件（请求事件）

这些事件需要**处理**（同意/拒绝）。

### FriendRequestEvent — 好友申请

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `User_Id` | long | 申请人 QQ |
| `Comment` | string | 验证信息 |
| `Flag` | string | 请求标识（用于处理请求） |

> ***示例 - 处理好友申请**：
```csharp
private async Task OnFriendRequest(FriendRequestEvent e)
{
    // 同意
    await Context.HandleFriendAddRequestAsync(e.Bot_Id, e.Flag, true, "");
    
    // 拒绝
    // await Context.HandleFriendAddRequestAsync(e.Bot_Id, e.Flag, false, "拒绝原因");
}
```

### GroupRequestEvent — 加群申请

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 目标群号 |
| `User_Id` | long | 申请人 QQ |
| `Comment` | string | 申请理由 |
| `Flag` | string | 请求标识 |

### BotGroupInviteEvent — 机器人被邀请加群

| 属性 | 类型 | 说明 |
|------|------|------|
| `Bot_Id` | long | 接收到消息的机器人 QQ |
| `Group_Id` | long | 邀请加入的群号 |
| `Operator_Id` | long | 邀请人 QQ |
| `Flag` | string | 请求标识 |

---

## System 事件（系统事件）

### PluginLoadedEvent - 插件被加载
| 属性 | 类型 | 说明 |
|------|------|------|
| `PluginId` | string | 被加载插件的唯一标识 ID |
| `PluginName` | string | 被加载插件的名称 |

### PluginUnloadedEvent - 插件被卸载
| 属性 | 类型 | 说明 |
|------|------|------|
| `PluginId` | string | 被卸载插件的唯一标识 ID |
| `PluginName` | string | 被卸载插件的名称 |

---

## 事件基类 EventBase

所有事件继承自 `EventBase`，包含两个通用属性：

```csharp
public class EventBase
{
    public DateTime Time { get; set; }           // 事件发生时间
    public IPluginContext Context { get; set; }   // 插件上下文
}
```

---
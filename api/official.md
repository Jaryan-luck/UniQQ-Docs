# 腾讯官方机器人 API

腾讯官方机器人能力通过 `Context.Official` 提供。这个入口只放官方机器人独有能力；普通文本、普通图片 URL、机器人列表、群列表、好友列表等能被 UniQQ 通用接口承载的能力，仍然优先使用 `IPluginContext`。

> 当前官方机器人适配处于测试阶段。实际可用范围还会受到腾讯官方开放平台账号权限、事件订阅、主动消息规则、群/单聊场景权限影响。腾讯官方消息接口支持文本、Markdown、Ark、Embed、富媒体等消息类型，且群聊、单聊使用 `group_openid` / `openid` 作为目标标识。

参考：[腾讯官方机器人发送消息文档](https://github.com/tencent-connect/bot-docs/blob/main/docs/develop/api-v2/server-inter/message/send-receive/send.md) 中的发送消息接口说明包含 `msg_type`：`0` 文本、`2` Markdown、`3` Ark、`4` Embed、`7` Media 富媒体，并分别使用 `/v2/groups/{group_openid}/messages` 与 `/v2/users/{openid}/messages` 发送群聊和单聊消息。

---

## 使用前判断机器人类型

```csharp
bool isOfficial = await Context.Official.IsOfficialBotAsync(botUin);

if (!isOfficial)
{
    await Context.WriteLog($"{botUin} 不是腾讯官方机器人");
    return;
}
```

如果插件只需要普通文本或普通图片 URL，不需要主动判断底层类型，直接调用通用接口即可。

```csharp
await Context.SendGroupMsgReturnIdAsync(
    botUin,
    groupId,
    MessageBuilder.Text("普通消息优先走通用接口"));
```

---

## 官方专属接口速查

| 接口 | 说明 |
|---|---|
| `Task<bool> IsOfficialBotAsync(long botUin)` | 判断指定机器人是否为正在运行的腾讯官方机器人 |
| `Task<OfficialMessageResult?> SendOfficialGroupMarkdownMessageAsync(long botUin, long groupId, string markdownContent, OfficialKeyboard? keyboard = null, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方群 Markdown 消息 |
| `Task<OfficialMessageResult?> SendOfficialPrivateMarkdownMessageAsync(long botUin, long userId, string markdownContent, OfficialKeyboard? keyboard = null, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方单聊 Markdown 消息 |
| `Task<OfficialMessageResult?> SendOfficialGroupArkMessageAsync(long botUin, long groupId, OfficialArkMessage ark, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方群 Ark 消息 |
| `Task<OfficialMessageResult?> SendOfficialPrivateArkMessageAsync(long botUin, long userId, OfficialArkMessage ark, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方单聊 Ark 消息 |
| `Task<OfficialMessageResult?> SendOfficialGroupEmbedMessageAsync(long botUin, long groupId, OfficialEmbedMessage embed, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方群 Embed 消息 |
| `Task<OfficialMessageResult?> SendOfficialPrivateEmbedMessageAsync(long botUin, long userId, OfficialEmbedMessage embed, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方单聊 Embed 消息 |
| `Task<OfficialMessageResult?> SendOfficialGroupMediaMessageAsync(long botUin, long groupId, string fileInfo, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方群富媒体消息，`fileInfo` 来自官方富媒体上传 |
| `Task<OfficialMessageResult?> SendOfficialPrivateMediaMessageAsync(long botUin, long userId, string fileInfo, string? eventId = null, string? replyMessageId = null, int? messageSequence = null)` | 发送官方单聊富媒体消息，`fileInfo` 来自官方富媒体上传 |
| `Task<OfficialMediaUploadResult?> UploadOfficialGroupMediaByUrlAsync(long botUin, long groupId, OfficialMediaType mediaType, string url, bool sendImmediately = false)` | 通过 URL 上传官方群富媒体 |
| `Task<OfficialMediaUploadResult?> UploadOfficialPrivateMediaByUrlAsync(long botUin, long userId, OfficialMediaType mediaType, string url, bool sendImmediately = false)` | 通过 URL 上传官方单聊富媒体 |
| `Task<bool> AcknowledgeOfficialInteractionAsync(long botUin, string interactionId, OfficialInteractionAckCode code = OfficialInteractionAckCode.Success)` | ACK 官方按钮交互事件 |
| `Task<OfficialIdMappingInfo?> GetOfficialIdMappingAsync(long botUin, OfficialIdScope scope, long virtualId)` | 查询 UniQQ 虚拟 ID 对应的官方 `openid` 或 `group_openid` |
| `Task<JObject?> CallOfficialOpenApiAsync(long botUin, string method, string path, JObject? body = null)` | 透传调用腾讯官方 OpenAPI |

---

## Markdown 消息

```csharp
await Context.Official.SendOfficialGroupMarkdownMessageAsync(
    botUin: e.Bot_Id,
    groupId: e.Group_Id,
    markdownContent: "# UniQQ\n这是一条官方 Markdown 消息");
```

如果需要按钮，可以构造 `OfficialKeyboard`：

```csharp
var keyboard = new OfficialKeyboard
{
    Content = new OfficialKeyboardContent
    {
        Rows =
        {
            new OfficialKeyboardRow
            {
                Buttons =
                {
                    new OfficialKeyboardButton
                    {
                        Id = "open_docs",
                        RenderData = new OfficialKeyboardButtonRenderData
                        {
                            Label = "查看文档",
                            VisitedLabel = "已查看",
                            Style = OfficialKeyboardButtonStyle.Blue
                        },
                        Action = new OfficialKeyboardButtonAction
                        {
                            Type = OfficialKeyboardActionType.LinkOrMiniApp,
                            Data = "https://uniqq-docs.jaryan.work",
                            Permission = new OfficialKeyboardPermission
                            {
                                Type = OfficialKeyboardPermissionType.Everyone
                            }
                        }
                    }
                }
            }
        }
    }
};

await Context.Official.SendOfficialGroupMarkdownMessageAsync(
    e.Bot_Id,
    e.Group_Id,
    "点击按钮查看 UniQQ 文档",
    keyboard);
```

---

## Ark 消息

```csharp
var ark = new OfficialArkMessage
{
    TemplateId = 23,
    KeyValues =
    {
        new OfficialArkKeyValue { Key = "#DESC#", Value = "UniQQ Ark 消息" },
        new OfficialArkKeyValue { Key = "#PROMPT#", Value = "来自 UniQQ 插件" }
    }
};

await Context.Official.SendOfficialGroupArkMessageAsync(
    e.Bot_Id,
    e.Group_Id,
    ark);
```

Ark 模板 ID 与字段取决于腾讯官方平台开放的模板和审核结果。插件应以平台当前允许的模板为准。

---

## Embed 消息

```csharp
var embed = new OfficialEmbedMessage
{
    Title = "UniQQ",
    Prompt = "插件通知",
    Thumbnail = new OfficialEmbedThumbnail
    {
        Url = "https://example.com/icon.png"
    },
    Fields =
    {
        new OfficialEmbedField { Name = "构建完成" },
        new OfficialEmbedField { Name = "插件已加载" }
    }
};

await Context.Official.SendOfficialPrivateEmbedMessageAsync(
    botUin,
    userId,
    embed);
```

---

## 官方富媒体

普通 URL 图片可以优先使用通用消息接口：

```csharp
var message = new Message();
message.Segments.Add(MessageBuilder.Text("图片："));
message.Segments.Add(MessageBuilder.Image("https://example.com/a.png"));

await Context.SendGroupMsgReturnIdAsync(botUin, groupId, message);
```

如果需要官方原生富媒体上传与发送，可以分两步：

```csharp
var upload = await Context.Official.UploadOfficialGroupMediaByUrlAsync(
    botUin,
    groupId,
    OfficialMediaType.Image,
    "https://example.com/a.png");

if (upload != null)
{
    await Context.Official.SendOfficialGroupMediaMessageAsync(
        botUin,
        groupId,
        upload.FileInfo);
}
```

`OfficialMediaType`：

| 值 | 说明 |
|---|---|
| `Image` | 图片 |
| `Video` | 视频 |
| `Voice` | 语音 |
| `File` | 文件 |

---

## 按钮交互 ACK

订阅官方交互事件后，如果插件处理了按钮点击，建议调用 ACK，避免客户端一直处于加载状态。

```csharp
public override Task OnEnable()
{
    Context.Events.On<InteractionCreateEvent>(OnInteraction);
    return Task.CompletedTask;
}

private async Task OnInteraction(InteractionCreateEvent e)
{
    await Context.Official.AcknowledgeOfficialInteractionAsync(
        e.Bot_Id,
        e.Interaction_Id,
        OfficialInteractionAckCode.Success);
}
```

`OfficialInteractionAckCode`：

| 值 | 说明 |
|---|---|
| `Success` | 成功 |
| `Failed` | 失败 |
| `TooFrequent` | 操作太频繁 |
| `Duplicated` | 重复操作 |
| `NoPermission` | 无权限 |
| `AdminOnly` | 仅管理员 |

---

## ID 映射

腾讯官方机器人不直接使用传统 QQ 号或群号作为接口目标，而是使用 `openid` 与 `group_openid`。UniQQ 会为插件层生成稳定的虚拟数字 ID，让旧插件仍能使用 `long userId`、`long groupId`。

如果插件需要调用官方原始 OpenAPI，可以先查询映射：

```csharp
var mapping = await Context.Official.GetOfficialIdMappingAsync(
    botUin,
    OfficialIdScope.Group,
    groupId);

if (mapping != null)
{
    await Context.WriteLog($"group_openid = {mapping.OfficialId}");
}
```

`OfficialIdScope`：

| 值 | 说明 |
|---|---|
| `User` | 用户 openid |
| `Group` | 群 group_openid |

---

## 原始 OpenAPI 调用

当 UniQQ 暂未封装某个官方能力，但腾讯官方 OpenAPI 已开放时，可以使用原始调用：

```csharp
var body = new JObject
{
    ["content"] = "通过官方 OpenAPI 发送",
    ["msg_type"] = 0
};

JObject? result = await Context.Official.CallOfficialOpenApiAsync(
    botUin,
    "POST",
    "/v2/groups/{group_openid}/messages",
    body);
```

注意事项：

- `path` 必须是腾讯官方 OpenAPI 路径或完整 URL。
- 插件作者需要自行确认官方接口所需字段、权限、限频与审核规则。
- 如果只是普通文本、普通图片 URL，请优先使用 UniQQ 通用接口，避免把插件写死到某一个底层。

---

## 官方专属事件

官方机器人会尽量转换为 UniQQ 通用事件：

| 官方事件 | UniQQ 事件 |
|---|---|
| `GROUP_MESSAGE_CREATE` / `GROUP_AT_MESSAGE_CREATE` | `GroupMessageEvent` |
| `C2C_MESSAGE_CREATE` | `PrivateMessageEvent` |
| `GROUP_ADD_ROBOT` | `BotJoinGroupEvent` |
| `GROUP_DEL_ROBOT` | `BotKickEvent` |
| `FRIEND_ADD` | `FriendAddedEvent` |
| `FRIEND_DEL` | `FriendDeletedEvent` |

官方独有事件：

| UniQQ 事件 | 说明 |
|---|---|
| `InteractionCreateEvent` | 官方按钮、指令等交互回调 |
| `MessageAuditEvent` | 官方消息审核结果 |
| `ActiveMessageSettingChangedEvent` | 用户或群开启/关闭主动消息接收能力 |

示例：

```csharp
Context.Events.On<MessageAuditEvent>(async e =>
{
    await Context.WriteLog(
        $"官方消息审核：{e.Official_Message_Id} => {(e.Passed ? "通过" : "拒绝")}");
});
```

---

## 返回模型

### OfficialMessageResult

| 属性 | 类型 | 说明 |
|---|---|---|
| `MessageId` | `string` | 腾讯官方消息 ID |
| `VirtualMessageId` | `long` | UniQQ 内部稳定虚拟消息 ID |
| `Timestamp` | `long` | 官方返回的发送时间 |
| `Raw` | `JObject` | 官方原始响应 |

### OfficialMediaUploadResult

| 属性 | 类型 | 说明 |
|---|---|---|
| `FileUuid` | `string` | 官方文件 UUID |
| `FileInfo` | `string` | 发送富媒体消息需要使用的 `file_info` |
| `Ttl` | `int` | 官方返回的有效期 |
| `MessageId` | `string` | 如果上传时直接发送，可能包含消息 ID |
| `Raw` | `JObject` | 官方原始响应 |

### OfficialIdMappingInfo

| 属性 | 类型 | 说明 |
|---|---|---|
| `Scope` | `OfficialIdScope` | 用户或群 |
| `VirtualId` | `long` | UniQQ 虚拟数字 ID |
| `OfficialId` | `string` | 官方 `openid` 或 `group_openid` |
| `DisplayName` | `string` | UniQQ 当前已知显示名称 |

---

## 推荐实践

1. 普通插件尽量只依赖通用接口。
2. 需要官方卡片、按钮、Markdown、富媒体上传时，再调用 `Context.Official`。
3. 官方机器人返回的群和用户 ID 是 UniQQ 虚拟 ID，不要把它当作真实 QQ 号或真实群号展示给用户。
4. 官方主动消息、被动回复、URL 配置、审核、限频等规则以腾讯开放平台返回结果为准。
5. 官方能力调用失败时，先查看 UniQQ 系统日志，日志会说明是否缺少映射、权限不足、接口不支持或官方平台返回错误。

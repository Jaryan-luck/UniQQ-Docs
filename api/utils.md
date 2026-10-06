# 工具函数

SDK 提供的 Message 构建、MessageBuilder 等实用工具。

---

## Message 类

`Message` 是 SDK 中表示一条消息的核心类。它既可以由消息段列表构成，也可以通过隐式转换从字符串快速创建。

### 属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `RawText` | `string` | 消息的纯文本内容（自动拼接所有 TextSegment） |

### 隐式转换

Message 支持从 `string` 隐式转换，简化纯文本消息的发送：

```csharp
// 以下两种写法等价
await Context.SendGroupMsgReturnIdAsync(bot, group, new Message { Segments = ... });
await Context.SendGroupMsgReturnIdAsync(bot, group, "这是一条纯文本消息");

// 也可以直接对 Message 对象赋值字符串
Message msg = "Hello World!";
```

### + 运算符

Message 支持 `+` 运算符拼接：

```csharp
Message msg = MessageBuilder.Text("你好，") + "欢迎入群！";
```

### 消息段列表

通过 `Segments` 属性手动构建复合消息：

```csharp
var msg = new Message
{
    Segments = new List<MessageSegment>
    {
        MessageBuilder.At(e.User_Id),
        MessageBuilder.Text(" 你好，"),
        MessageBuilder.Face(76),
    }
};
await Context.SendGroupMsgReturnIdAsync(e.Bot_Id, e.Group_Id, msg);
```

---

## MessageSegment — 消息段抽象基类

所有消息段继承自 `MessageSegment`。可用段类型：

| 类 | 说明 |
|---|---|
| `TextSegment` | 纯文本 |
| `AtSegment` | @某人的消息 |
| `ImageSegment` | 图片 |
| `FaceSegment` | QQ 表情 |
| `ReplySegment` | 引用回复 |
| `VoiceSegment` | 语音 |
| `VideoSegment` | 视频 |
| `JsonSegment` | JSON 卡片 |
| `XmlSegment` | XML 卡片 |
| `MusicSegment` | 音乐分享 |
| `ShareSegment` | 链接分享 |
| `FileSegment` | 文件 |

---

## MessageBuilder — 静态工厂类

`using UniQQ.SDK.Builders;`

MessageBuilder 提供快速创建各种消息段的静态方法。

### 基础消息段

```csharp
// 纯文本
MessageBuilder.Text(string text)

// @成员（0 = @全体成员）
MessageBuilder.At(long target)

// QQ 表情（表情 ID，如 76 = 爱心）
MessageBuilder.Face(int faceId)
```

### 图片

```csharp
// 本地文件路径
MessageBuilder.Image(string filePath)

// 网络 URL
MessageBuilder.ImageByUrl(string url)
```

### 引用回复

```csharp
// 引用回复某条消息
MessageBuilder.Reply(long messageId)
```

### 语音 / 视频

```csharp
MessageBuilder.Voice(string filePath)
MessageBuilder.Video(string filePath)
```

### JSON / XML 卡片

```csharp
MessageBuilder.Json(string jsonContent)
MessageBuilder.Xml(string xmlContent)
```

### 音乐分享

```csharp
// QQ音乐
MessageBuilder.QQMusic(string songId)

// 网易云音乐
MessageBuilder.NeteaseMusic(string songId)
```

### 链接分享

```csharp
MessageBuilder.Share(string url, string title, string? description = null, string? imageUrl = null)
```

### 文件

```csharp
MessageBuilder.File(string filePath)
```

---

## 快捷回复 — ReplyAsync

事件对象（如 `GroupMessageEvent`）提供 `ReplyAsync` 便捷方法：

```csharp
// e 是 GroupMessageEvent
// Reference: true 表示引用原消息，false 则直接回复
await e.ReplyAsync(MessageBuilder.Text("收到"), Reference: true);
```

---

## PluginManifest — 插件清单模型

`Context.Manifest` 返回的 `PluginManifest` 提供以下字段（与 `plugin.json` 对应）：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Id` | `string` | 插件唯一标识 |
| `Name` | `string` | 显示名称 |
| `Author` | `string` | 作者 |
| `Version` | `string` | 版本号 |
| `Description` | `string` | 描述 |
| `Runtime` | `string` | 运行时类型（dotnet） |
| `Entry` | `string` | 入口 DLL |
| `SdkVersion` | `string` | SDK 版本 |
| `MinFrameworkVersion` | `string` | 最低框架版本 |
| `Architecture` | `string` | 架构兼容性 |
| `Website` | `string` | 官网 URL |

---

## 使用 Context.DataPath 做持久化

`Context.DataPath` 是框架为每个插件分配的专属数据目录：`{UniQQ}/Data/plugins/{pluginId}/`

```csharp
public override Task OnEnable()
{
    // 确保目录存在
    Directory.CreateDirectory(Context.DataPath);

    // 读取配置文件
    string configPath = Path.Combine(Context.DataPath, "config.json");
    if (File.Exists(configPath))
    {
        string json = File.ReadAllText(configPath);
        // ...
    }
}
```

### 最佳实践

1. **使用 JSON 格式存储配置**：`JsonSerializer`（System.Text.Json）或 `Newtonsoft.Json`
2. **按需创建子目录**：`logs/`、`cache/`、`backup/`
3. **异步 IO**：使用 `File.ReadAllTextAsync` / `WriteAllTextAsync` 避免阻塞
4. **不要在 Load 阶段访问**：`Context.DataPath` 仅在 OnEnable 之后可用

---

## 数据模型速查

### 直接来看文档
或者 
SDK 定义了以下数据模型（位于 `UniQQ.SDK.Models` 命名空间）：

| 模型 | 说明 |
|---|---|
| `Bot` | 机器人信息（Uin / NickName / AvatarUrl / OnlineTime） |
| `Friend` | 好友信息（Uin / NickName / Remark / Age / Sex / Level / AvatarUrl / Uid / BirthdayYear / BirthdayMonth / BirthdayDay / PhoneNum / Email / CategoryId / CategoryName / QId / LoginDays） |
| `Group` | 群信息（Uin / GroupName / Remark / MemberCount / MaxMemberCount / GroupAllShut / AvatarUrl） |
| `GroupMember` | 群成员（Uin / NickName / CardName / AvatarUrl / Role / Title / JoinTime / LastSentTime / ShutUpTimestamp / QQLevel / GroupLevel / Sex / Age / Area / QAge / IsRobot / Unfriendly / CardChangeable / TitleExpireTime） |
| `GroupNotice` | 群公告（NoticeId / SenderUin / PublishTime / Message / Title / PublishDateTime） |
| `GroupFileSystemInfo` | 群文件空间（FileCount / LimitCount / UsedSpace / TotalSpace / TotalSizeReadable） |
| `GroupFile` | 群文件（FileId / FileName / FileSize / UploaderUin / UploadTime / IsFolder / UploadDateTime / FileSizeReadable） |
| `EssenceMessage` | 精华消息（MessageId / SenderUin / SenderNickName / SenderCard / AddTime / OperatorUin / OperatorNickName / Content / AddDateTime） |
| `ForwardMessageNode` | 合并转发消息节点（MessageId / UserId / Nickname / Content / RawMessage / Time / RawData / MessageTime） |
| `ForwardNode` | 合并转发节点数据模型（UserId / Nickname / Content） |
| `GroupHonor` | 群荣誉项（Uin / NickName / AvatarUrl / Description / DayCount） |
| `GroupHonorInfo` | 群荣誉信息（GroupUin / Talkative / Performers / Legend / Strong） |
| `GroupAtAllRemainInfo` | @全体 剩余（GroupUin / CanAtAll / RemainCount / RemainCountForUin） |
| `ShutUpMember` | 被禁言成员（Uin / NickName / ShutUpTimestamp / ShutUpEndTime / IsShutUp） |
| `OneWayFriend` | 单向好友（Uin / NickName / AvatarUrl / Source / AddTime / AddDateTime） |
| `OcrResult` | OCR 识别结果（Texts / FullText / Language / HasText） |
| `ImageDetail` | 图片详情（FileId / Url / FileSize / FileName / Width / Height / FileSizeReadable） |
| `RecordDetail` | 语音详情（FileId / Url / FileSize / FileName / Duration / FileSizeReadable） |
| `VideoDetail` | 视频详情（FileId / Url / FileSize / FileName / Width / Height / Duration / ThumbUrl / FileSizeReadable） |
| `PrivateFileInfo` | 私聊文件详情（FileId / FileName / Url / FileSize / FileSizeReadable） |
| `MessageDetail` | 消息详情（MessageId / RealId / MessageType / UserId / GroupId / SenderId / SenderNickname / SenderCard / Message / RawMessage / Time / SubType / Font / MessageTime / IsGroupMessage / IsPrivateMessage） |
| `Sender` | 发送者（Uin / NickName） |
| `FriendAddRequest` | 好友申请（RequestId / Flag / RequesterUin / RequesterNickName / Message / RequestTime / IsIgnored / RequestDateTime / Summary） |
| `GroupAddRequest` | 加群申请（RequestId / Flag / RequesterUin / RequesterNickName / Message / GroupUin / GroupName / RequestTime / IsIgnored / RequestDateTime / Summary） |
| `UniQQApplicationInfo` | UniQQ 应用信息（AppName / Version） |

> 注：以上模型定义在 `UniQQ.Core.dll` 中，SDK 通过接口引用。

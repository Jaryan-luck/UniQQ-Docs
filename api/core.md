# 核心 API

本文档记录 UniQQ.SDK 当前对插件公开的核心接口。UniQQ 现在支持两类机器人底层：

- **第三方机器人**：当前由 NapCat 接入，能力最完整，适合需要 QQ 客户端原生能力的插件。
- **腾讯官方机器人**：当前为测试阶段，遵循腾讯 QQ 机器人开放平台能力边界，普通消息和部分查询能力会尽量兼容 UniQQ 通用接口，官方独有能力通过 `Context.Official` 调用。

> 建议插件优先使用 `IPluginContext` 中的通用接口。只有 Markdown、Ark、Embed、官方富媒体上传、按钮 ACK、官方 OpenAPI 透传等官方独有能力，才使用 `Context.Official`。

---

## 命名空间总览

| 命名空间 | 用途 |
|---|---|
| `UniQQ.SDK.Plugins` | 插件基类 `PluginBase` 与插件清单模型 |
| `UniQQ.SDK.Interfaces` | `IPluginContext`、`IEventBus`、`IOfficialBotApi` |
| `UniQQ.SDK.Events.*` | 消息、通知、请求、元事件、系统事件 |
| `UniQQ.SDK.Models` | `Bot`、`Group`、`Friend`、`Message`、`GroupMember` 等通用模型 |
| `UniQQ.SDK.Models.Segments` | 文本、@、图片、语音、视频、回复、转发、JSON、XML 等消息段 |
| `UniQQ.SDK.Official` | 腾讯官方机器人专属消息、按钮、富媒体、ID 映射等模型 |
| `UniQQ.SDK.Builders` | `MessageBuilder` 消息构建器 |
| `UniQQ.SDK.Enums` | `MemberRole`、`MessageType`、`BotStatus`、`OnlineStatus` 等枚举 |

---

## PluginBase

所有插件都继承 `PluginBase`。

```csharp
using UniQQ.SDK.Plugins;

public abstract class PluginBase : IPlugin
{
    public IPluginContext Context { get; }

    public abstract string Name { get; }
    public abstract string Version { get; }

    public virtual Task Load() => Task.CompletedTask;
    public virtual Task OnEnable() => Task.CompletedTask;
    public virtual Task OnDisable() => Task.CompletedTask;
    public virtual Task OnSettings() => Task.CompletedTask;
    public virtual Task Unload() => Task.CompletedTask;
}
```

`Context` 在 `OnEnable()`、`OnDisable()`、`OnSettings()` 和事件处理方法中可用。不要在 `Load()` 阶段调用需要运行时上下文的 UniQQ API。

---

## IPluginContext 属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Manifest` | `PluginManifest` | 当前插件的清单信息 |
| `DataPath` | `string` | 当前插件独立数据目录 |
| `Events` | `IEventBus` | 插件事件总线 |
| `Official` | `IOfficialBotApi` | 腾讯官方机器人专属能力入口 |

---

## 兼容性标记

| 标记 | 含义 |
|---|---|
| 通用 | 第三方机器人和腾讯官方机器人都可以调用 |
| 通用但受限 | 接口可调用，但腾讯官方机器人受官方平台限制，数据可能来自事件缓存或能力降级 |
| 第三方专用 | 依赖 NapCat / QQ 客户端能力，腾讯官方机器人不支持 |
| 官方专用 | 只服务腾讯官方机器人，通过 `Context.Official` 调用 |

腾讯官方机器人调用不支持的通用接口时，UniQQ 会写入系统日志，并返回该接口对应的失败值，例如 `false`、`null`、`0`、空数组或空列表。这样可以尽量避免旧插件因为底层能力不足直接崩溃。

---

## 运行与底层调用

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<bool> OnlineBotAsync(long botUin)` | 通用 | 上线指定机器人。根据机器人类型自动走第三方或官方登录链路 |
| `Task<bool> OfflineBotAsync(long botUin)` | 通用 | 下线指定机器人 |
| `Task<List<Bot>?> GetBotListAsync()` | 通用 | 获取 UniQQ 当前机器人列表与在线状态 |
| `Task<Bot?> GetLoginBotInfoAsync(long botUin)` | 通用 | 获取指定在线机器人自身信息 |
| `Task<UniQQApplicationInfo> UniQQApplicationInfo()` | 通用 | 获取 UniQQ 程序信息 |
| `Task<JObject?> Call_ApiAsync(long botUin, string action, JObject? parameters = null)` | 第三方优先 | 第三方机器人可用于调用 NapCat / OneBot action；官方机器人只接受官方 OpenAPI 路径，建议官方原始调用使用 `Context.Official.CallOfficialOpenApiAsync` |

---

## 消息发送

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<long[]> SendGroupMsgReturnIdAsync(long botUin, long groupId, Message message)` | 通用但受限 | 发送群消息并返回消息 ID。官方机器人支持文本、图片 URL、部分 @ 能力 |
| `Task<long> SendPrivateMsgReturnIdAsync(long botUin, long userId, Message message)` | 通用但受限 | 发送私聊消息并返回消息 ID。官方机器人支持文本、图片 URL |
| `Task<long> SendGroupTempMsgReturnIdAsync(long botUin, long groupUin, long userId, Message message)` | 第三方专用 | 发送群临时会话消息 |
| `Task<long> SendGroupForwardMsgAsync(long botUin, long groupId, IEnumerable<ForwardNode>? nodes)` | 第三方专用 | 发送群合并转发 |
| `Task<long> SendPrivateForwardMsgAsync(long botUin, long userId, IEnumerable<ForwardNode>? nodes)` | 第三方专用 | 发送私聊合并转发 |
| `Task SendGroupMessageAsync(long botUin, long groupId, Message message)` | 通用但受限 | 旧版兼容接口，建议改用 `SendGroupMsgReturnIdAsync` |
| `Task SendPrivateMessageAsync(long botUin, long userId, Message message)` | 通用但受限 | 旧版兼容接口，建议改用 `SendPrivateMsgReturnIdAsync` |
| `Task SendGroupTempMessageAsync(long botUin, long groupUin, long userId, Message message)` | 第三方专用 | 旧版兼容接口，建议改用 `SendGroupTempMsgReturnIdAsync` |
| `Task<bool> SetMessageEmojiLikeAsync(long botUin, long messageId, string emojiId)` | 第三方专用 | 为指定消息设置或取消表情回应 |

### 消息段兼容说明

| 消息段 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `TextSegment` | 完整支持 | 支持 |
| `AtSegment` | 支持 @成员 与 @全体 | 支持已建立映射的成员 @；@全体会降级为文本 `@全体成员` |
| `ImageSegment` | 支持本地文件、URL、file_id 等 NapCat 能力 | 支持 http/https 图片 URL；本地图片暂不直接发送 |
| `ReplySegment` | 支持 | 当前通用发送中忽略；官方原生消息可用 `replyMessageId` |
| `FaceSegment` | 支持 | 降级为文本 `[表情:id]` |
| `VoiceSegment` / `VideoSegment` / `FileSegment` | 支持 | 普通通用发送会降级；官方专属富媒体请使用 `Context.Official` |
| `JsonSegment` / `XmlSegment` | 支持 | 降级为“不支持”文本 |
| `ForwardSegment` | 支持读取与发送合并转发 | 官方开放平台无等价能力 |
| `MusicSegment` / `ShareSegment` | 支持 | 降级为普通类型标记文本 |

---

## 账号资料与凭证

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<bool> SetBotInfoAsync(long botUin, string? nickname = null, int? gender = null, string? birthday = null, string? location = null, string? personalNote = null)` | 第三方专用 | 修改账号资料 |
| `Task<bool> SetBotAvatarAsync(long botUin, string imagePath)` | 第三方专用 | 修改机器人头像 |
| `Task<bool> SetBotSignatureAsync(long botUin, string longnick)` | 第三方专用 | 修改个性签名 |
| `Task<Dictionary<string, string>?> GetCookiesAsync(long botUin, string? domain = null)` | 第三方专用 | 获取 QQ 客户端 Cookies |
| `Task<string?> GetCsrfTokenAsync(long botUin)` | 第三方专用 | 获取 CSRF Token |
| `Task<Dictionary<string, object>?> GetCredentialsAsync(long botUin, string domain = "qun.qq.com")` | 第三方专用 | 获取客户端凭证 |
| `Task<string?> GetRKeyAsync(long botUin)` | 第三方专用 | 获取资源 RKey |
| `Task<string?> GetClientKeyAsync(long botUin)` | 第三方专用 | 获取 ClientKey |

---

## 好友与单聊

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<List<Friend>?> GetFriendListAsync(long botUin)` | 通用但受限 | 第三方返回好友列表；官方返回事件缓存中的已知单聊用户 |
| `Task<Friend?> GetFriendInfoAsync(long botUin, long FriendUin)` | 通用但受限 | 第三方读取好友资料；官方只能返回已知映射用户 |
| `Task<bool> SetFriendRemarkAsync(long botUin, long friendUin, string remark)` | 第三方专用 | 设置好友备注 |
| `Task<bool> DeleteFriendAsync(long botUin, long friendUin)` | 第三方专用 | 删除好友 |
| `Task<bool> SendFriendAddRequestAsync(long botUin, long targetUin, string? message = null)` | 第三方专用 | 主动添加好友 |
| `Task<bool> SendLikeAsync(long botUin, long friendUin, int count = 1)` | 第三方专用 | 好友点赞 |
| `Task<bool> SendPrivatePokeAsync(long botUin, long targetUin)` | 第三方专用 | 私聊戳一戳 |
| `Task<List<OneWayFriend>?> GetOneWayFriendListAsync(long botUin)` | 第三方专用 | 获取单向好友列表 |

---

## 群与成员

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<List<Group>?> GetGroupListAsync(long botUin)` | 通用但受限 | 第三方返回群列表；官方返回事件缓存中的已知群 |
| `Task<Group?> GetGroupInfoAsync(long botUin, long groupUin)` | 通用但受限 | 第三方读取真实群资料；官方返回已知群或虚拟群信息 |
| `Task<List<GroupMember>?> GetGroupMemberListAsync(long botUin, long groupUin)` | 通用但受限 | 第三方返回群成员列表；官方返回事件缓存中的已知成员 |
| `Task<GroupMember?> GetGroupMemberInfoAsync(long botUin, long groupUin, long memberUin)` | 通用但受限 | 第三方读取成员资料；官方返回已知成员 |
| `Task<string> GetUserAvatarUrl(long userId, int size = 640)` | 通用 | 拼接 QQ 头像 URL |
| `Task<string> GetGroupAvatarUrl(long groupId, int size = 640)` | 通用 | 拼接群头像 URL |

官方机器人使用 `openid` / `group_openid`。为了让旧插件继续使用 `long` 类型的 `User_Id`、`Group_Id`，UniQQ 会为官方 ID 生成稳定的虚拟数字 ID，并在事件和查询接口中使用这些虚拟 ID。

---

## 群管理

以下能力依赖 QQ 客户端或 NapCat 能力，腾讯官方机器人当前不支持等价通用能力。

| 接口 | 说明 |
|---|---|
| `Task SetGroupMemberMuteAsync(long botUin, long groupUin, long memberUin, int durationSeconds)` | 设置成员禁言或解除禁言 |
| `Task SetGroupWholeMuteAsync(long botUin, long groupUin, bool enable)` | 设置全员禁言 |
| `Task SetGroupAdminAsync(long botUin, long groupUin, long memberUin, bool isAdmin)` | 设置或取消管理员 |
| `Task RecallMessageAsync(long botUin, long messageId)` | 撤回消息 |
| `Task KickGroupMemberAsync(long botUin, long groupUin, long memberUin, bool rejectAddRequest = false)` | 踢出群成员 |
| `Task LeaveGroupAsync(long botUin, long groupUin, bool isDismiss = false)` | 退出群或解散群 |
| `Task<bool> SetGroupMemberTitleAsync(long botUin, long groupUin, long memberUin, string title)` | 设置专属头衔 |
| `Task<bool> SetGroupMemberCardAsync(long botUin, long groupUin, long memberUin, string card)` | 设置群名片 |
| `Task<bool> SetGroupNameAsync(long botUin, long groupUin, string newName)` | 修改群名称 |
| `Task<bool> SetGroupRemarkAsync(long botUin, long groupUin, string remark)` | 设置群备注 |
| `Task<bool> SetGroupPortraitAsync(long botUin, long groupUin, string imagePath)` | 设置群头像 |
| `Task<bool> SendGroupPokeAsync(long botUin, long groupUin, long targetUin)` | 群戳一戳 |
| `Task<List<ShutUpMember>?> GetGroupShutListAsync(long botUin, long groupUin)` | 获取禁言列表 |
| `Task<GroupHonorInfo?> GetGroupHonorInfoAsync(long botUin, long groupUin)` | 获取群荣誉 |
| `Task<GroupAtAllRemainInfo?> GetGroupAtAllRemainAsync(long botUin, long groupUin)` | 获取 @全体 剩余次数 |

---

## 请求处理

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<List<FriendAddRequest>?> GetFriendAddRequestsAsync(long botUin)` | 第三方专用 | 获取好友申请列表 |
| `Task<bool> HandleFriendAddRequestAsync(long botUin, string flag, bool approve, string? remark = null)` | 第三方专用 | 处理好友申请 |
| `Task<List<FriendAddRequest>?> GetIgnoredFriendAddRequestsAsync(long botUin)` | 第三方专用 | 获取已忽略好友申请 |
| `Task<bool> HandleIgnoredFriendAddRequestAsync(long botUin, string flag, bool approve, string? remark = null)` | 第三方专用 | 处理已忽略好友申请 |
| `Task<List<GroupAddRequest>?> GetGroupAddRequestsAsync(long botUin, long? groupUin = null)` | 第三方专用 | 获取加群申请 |
| `Task<bool> HandleGroupAddRequestAsync(long botUin, string flag, bool approve, string? reason = null)` | 第三方专用 | 处理单个加群申请 |
| `Task<int> HandleGroupAddRequestsAsync(long botUin, List<GroupAddRequest> requests, bool approve, string? reason = null)` | 第三方专用 | 批量处理加群申请 |
| `Task<List<GroupAddRequest>?> GetIgnoredGroupAddRequestsAsync(long botUin, long? groupUin = null)` | 第三方专用 | 获取已忽略加群申请 |
| `Task<bool> HandleIgnoredGroupAddRequestAsync(long botUin, string flag, bool approve, string? reason = null)` | 第三方专用 | 处理已忽略加群申请 |
| `Task<bool> SendGroupAddRequestAsync(long botUin, long groupUin, string? message = null)` | 第三方专用 | 主动申请加群 |

---

## 群文件、公告与精华

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<GroupFileSystemInfo?> GetGroupFileSystemInfoAsync(long botUin, long groupUin)` | 第三方专用 | 获取群文件系统信息 |
| `Task<List<GroupFile>?> GetGroupRootFilesAsync(long botUin, long groupUin)` | 第三方专用 | 获取群根目录文件 |
| `Task<List<GroupFile>?> GetGroupFilesByFolderAsync(long botUin, long groupUin, string folderId)` | 第三方专用 | 获取群文件夹内容 |
| `Task<bool> UploadGroupFileAsync(long botUin, long groupUin, string filePath, string? fileName = null, string? folderId = null)` | 第三方专用 | 上传群文件 |
| `Task<bool> DeleteGroupFileAsync(long botUin, long groupUin, string fileId)` | 第三方专用 | 删除群文件 |
| `Task<bool> CreateGroupFileFolderAsync(long botUin, long groupUin, string folderName)` | 第三方专用 | 创建群文件夹 |
| `Task<bool> DeleteGroupFolderAsync(long botUin, long groupUin, string folderId)` | 第三方专用 | 删除群文件夹 |
| `Task<string?> GetGroupFileUrlAsync(long botUin, long groupUin, string fileId)` | 第三方专用 | 获取群文件下载链接 |
| `Task<bool> RenameGroupFileAsync(long botUin, long groupUin, string fileId, string newName)` | 第三方专用 | 重命名群文件 |
| `Task<bool> MoveGroupFileAsync(long botUin, long groupUin, string fileId, string targetFolderId = "")` | 第三方专用 | 移动群文件 |
| `Task<bool> TransGroupFileAsync(long botUin, long groupUin, string fileId, int busId)` | 第三方专用 | 转存临时群文件 |
| `Task<bool> UploadPrivateFileAsync(long botUin, long userId, string filePath, string? fileName = null)` | 第三方专用 | 上传私聊文件 |
| `Task<PrivateFileInfo?> GetPrivateFileUrlAsync(long botUin, string fileId)` | 第三方专用 | 获取私聊文件下载信息 |
| `Task<List<GroupNotice>?> GetGroupNoticeListAsync(long botUin, long groupUin)` | 第三方专用 | 获取群公告列表 |
| `Task<bool> SendGroupNoticeAsync(long botUin, long groupUin, string title, string content)` | 第三方专用 | 发送群公告 |
| `Task<bool> DeleteGroupNoticeAsync(long botUin, long groupUin, string noticeId)` | 第三方专用 | 删除群公告 |
| `Task<bool> SetEssenceMessageAsync(long botUin, long messageId)` | 第三方专用 | 设置精华消息 |
| `Task<bool> DeleteEssenceMessageAsync(long botUin, long messageId)` | 第三方专用 | 移除精华消息 |
| `Task<List<EssenceMessage>?> GetEssenceMessageListAsync(long botUin, long groupUin)` | 第三方专用 | 获取精华消息列表 |

---

## 消息详情、历史与媒体获取

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<MessageDetail?> GetMessageAsync(long botUin, long messageId)` | 第三方专用 | 按消息 ID 获取消息详情 |
| `Task<List<ForwardMessageNode>?> GetForwardMsgAsync(long botUin, string messageId)` | 第三方专用 | 获取合并转发内容 |
| `Task<List<MessageDetail>?> GetGroupMessageHistoryAsync(long botUin, long groupUin, int count = 20, long? messageSeq = null)` | 第三方专用 | 获取群历史消息 |
| `Task<List<MessageDetail>?> GetFriendMessageHistoryAsync(long botUin, long userId, int count = 20, long? messageSeq = null)` | 第三方专用 | 获取私聊历史消息 |
| `Task<ImageDetail?> GetImageAsync(long botUin, string fileId)` | 第三方专用 | 获取图片详情或下载链接 |
| `Task<RecordDetail?> GetRecordAsync(long botUin, string fileId)` | 第三方专用 | 获取语音详情 |
| `Task<VideoDetail?> GetVideoAsync(long botUin, string fileId)` | 第三方专用 | 获取视频详情 |
| `Task<OcrResult?> OcrImageAsync(long botUin, string imageUrl)` | 第三方专用 | OCR 识别图片 |
| `Task<OcrResult?> OcrImageFromFileAsync(long botUin, string imagePath)` | 第三方专用 | OCR 识别本地图片 |

官方机器人事件中的附件会在事件解析时尽量转换为 `ImageSegment`、`VoiceSegment`、`VideoSegment` 或 `FileSegment`。官方平台没有与 NapCat `file_id` 二次查询等价的通用接口。

---

## 插件自管理与日志

| 接口 | 兼容性 | 说明 |
|---|---|---|
| `Task<bool> ReloadThisPlugin()` | 通用 | 重载当前插件 |
| `Task<bool> WriteLog(string logContent, Color logColor = default)` | 通用 | 输出日志到 UniQQ 日志面板 |

---

## 事件总线

```csharp
public interface IEventBus
{
    void On<TEvent>(Func<TEvent, Task> handler) where TEvent : EventBase;
    void Off<TEvent>() where TEvent : EventBase;
}
```

示例：

```csharp
public override Task OnEnable()
{
    Context.Events.On<GroupMessageEvent>(OnGroupMessage);
    Context.Events.On<PrivateMessageEvent>(OnPrivateMessage);
    return Task.CompletedTask;
}

public override Task OnDisable()
{
    Context.Events.Off<GroupMessageEvent>();
    Context.Events.Off<PrivateMessageEvent>();
    return Task.CompletedTask;
}
```

---

## 常用示例

### 通用文本回复

```csharp
private async Task OnGroupMessage(GroupMessageEvent e)
{
    if (e.Message.RawText.Trim() == "hello")
    {
        await Context.SendGroupMsgReturnIdAsync(
            e.Bot_Id,
            e.Group_Id,
            MessageBuilder.Text("Hello!"));
    }
}
```

### 通用 @ 回复

```csharp
private async Task OnGroupMessage(GroupMessageEvent e)
{
    var message = new Message();
    message.Segments.Add(MessageBuilder.At(e.User_Id));
    message.Segments.Add(MessageBuilder.Text(" 收到"));

    await Context.SendGroupMsgReturnIdAsync(e.Bot_Id, e.Group_Id, message);
}
```

官方机器人在发送 @ 时需要已经存在 UniQQ 虚拟用户 ID 与官方 `openid` 的映射。这个映射通常来自用户发消息、按钮交互、加好友、入群等事件。

---

## 下一步

- [官方机器人 API](/api/official)：查看 `Context.Official` 专属接口。
- [底层能力差异表](/api/bot-compatibility)：查看第三方机器人与腾讯官方机器人的完整差异。
- [事件系统](/concepts/events)：查看事件订阅方式。
- [工具函数](/api/utils)：查看 `MessageBuilder` 与消息段用法。

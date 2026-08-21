# 核心 API 参考

本文档详细列出 UniQQ.SDK 中所有供插件开发者使用的公开 API。

---

## 命名空间总览

| 命名空间 | 用途 |
|---|---|
| `UniQQ.SDK.Plugins` | 插件基类 `PluginBase` |
| `UniQQ.SDK.Interfaces` | `IPluginContext`、`IEventBus` 接口 |
| `UniQQ.SDK.Events.*` | 所有事件类型（Message / Notice / Request / Meta / System） |
| `UniQQ.SDK.Models` | 数据模型（Group / Friend / Bot / GroupMember 等） |
| `UniQQ.SDK.Models.GroupFile` | 群文件相关模型 |
| `UniQQ.SDK.Models.Segments` | 消息段（Text / At / Image / Face / Reply 等） |
| `UniQQ.SDK.Builders` | `MessageBuilder` 静态工厂类 |
| `UniQQ.SDK.Enums` | 枚举（MemberRole / MessageType / BotStatus / OnlineStatus） |

---

## PluginBase — 插件基类

所有插件必须继承 `PluginBase`。

```csharp
using UniQQ.SDK.Plugins;

public abstract class PluginBase : IPlugin
{
    public IPluginContext Context { get; }   // 插件上下文（OnEnable 之后可用）

    public abstract string Name { get; }     // 插件显示名称
    public abstract string Version { get; }  // 插件版本号

    public virtual Task Load()       => Task.CompletedTask;  // 加载阶段（禁止调用 Context）
    public virtual Task OnEnable()   => Task.CompletedTask;  // 启用阶段（订阅事件）
    public virtual Task OnDisable()  => Task.CompletedTask;  // 禁用阶段（取消订阅）
    public virtual Task OnSettings() => Task.CompletedTask;  // 设置窗口
    public virtual Task Unload()     => Task.CompletedTask;  // 卸载清理
}
```

**Context 可用阶段**：`OnEnable()` / `OnDisable()` / `OnSettings()` / 事件处理方法。在 `Load()` 和 `Unload()` 阶段不可用。

---

## IPluginContext — 插件上下文

插件通过 `Context` 调用所有 UniQQ 能力。以下按功能域分类。

### 插件生命周期与底层 API

| 方法签名 | 说明 |
|---|---|
| `Task<bool> ReloadThisPlugin()` | 重新载入本插件 |
| `Task<JObject?> Call_ApiAsync(long botUin, string action, JObject? parameters = null)` | 发送原始 API 请求 |

### 机器人账号控制与信息设置

| 方法签名 | 说明 |
|---|---|
| `Task<bool> OnlineBotAsync(long botUin)` | 上线机器人 |
| `Task<bool> OfflineBotAsync(long botUin)` | 下线机器人 |
| `Task<bool> SetBotInfoAsync(long botUin, string? nickname = null, int? gender = null, string? birthday = null, string? location = null, string? personalNote = null)` | 设置账号信息（昵称、性别、生日等） |
| `Task<bool> SetBotAvatarAsync(long botUin, string imagePath)` | 设置头像；imagePath 支持本地文件路径或网络 URL |
| `Task<bool> SetBotSignatureAsync(long botUin, string longnick)` | 设置个性签名 |
| `Task<Bot?> GetLoginBotInfoAsync(long botUin)` | 	获取当前登录的机器人信息 |
| `Task<List<Bot>?> GetBotListAsync()` | 获取所有已登录机器人列表 |
| `Task<UniQQApplicationInfo> UniQQApplicationInfo()` | 获取 UniQQ 应用自身信息 |

### 凭证、Cookie、Rkey 等底层凭证接口

| 方法签名 | 说明 |
|---|---|
| `Task<Dictionary<string, string>?> GetCookiesAsync(long botUin, string? domain = null)` | 获取 Cookies，不传 domain 获取全部 Cookies |
| `Task<string?> GetCsrfTokenAsync(long botUin)` | 获取 CSRF Token |
| `Task<Dictionary<string, object>?> GetCredentialsAsync(long botUin, string domain = "qun.qq.com")` | 获取通用凭证字典 |
| `Task<string?> GetRKeyAsync(long botUin)` | 获取 Rkey（资源鉴权密钥） |
| `Task<string?> GetClientKeyAsync(long botUin)` | 获取 ClientKey（客户端密钥） |

### 消息发送

| 方法签名 | 说明 |
|---|---|
| `Task SendGroupMessageAsync(long botUin, long groupId, Message message)` | 发送群消息 |
| `Task<long[]> SendGroupMsgReturnIdAsync(long botUin, long groupId, Message message)` | 发送群消息，返回message_id列表 |
| `Task SendPrivateMessageAsync(long botUin, long userId, Message message)` | 发送私聊消息 |
| `Task<long> SendPrivateMsgReturnIdAsync(long botUin, long userId, Message message)` | 发送私聊消息，返回message_id |
| `Task SendGroupTempMessageAsync(long botUin, long groupUin, long userId, Message message)` | 发送群临时会话消息 |
| `Task<long> SendGroupTempMsgReturnIdAsync(long botUin, long groupUin, long userId, Message message)` | 发送群临时会话消息，返回message_id |
| `Task<long> SendGroupForwardMsgAsync(long botUin, long groupId, IEnumerable<ForwardNode>? nodes)` | 发送群聊合并转发消息，返回message_id |
| `Task<long> SendPrivateForwardMsgAsync(long botUin, long userId, IEnumerable<ForwardNode>? nodes)` | 发送私聊合并转发消息，返回message_id |
| `Task<bool> SetMessageEmojiLikeAsync(long botUin, long messageId, string emojiId)` | 为消息设置 / 取消表情回应（贴表情），emojiId 例如 "181" |

**示例**：

```csharp
// 文本消息
await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id, MessageBuilder.Text("Hello!"));

// @某人 + 文字
await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id,
    new Message { Segments = new List<MessageSegment> {
        MessageBuilder.At(e.User_Id),
        MessageBuilder.Text(" 欢迎！")
    }});
```

### 好友操作

| 方法签名 | 说明 |
|---|---|
| `Task<List<Friend>?> GetFriendListAsync(long botUin)` | 获取好友列表 |
| `Task<Friend?> GetFriendInfoAsync(long botUin, long friendUin)` | 获取好友 / 陌生人详细信息 |
| `Task<bool> SetFriendRemarkAsync(long botUin, long friendUin, string remark)` | 设置好友备注 |
| `Task<bool> DeleteFriendAsync(long botUin, long friendUin)` | 删除好友 |
| `Task<bool> SendFriendAddRequestAsync(long botUin, long targetUin, string? message = null)` | Bot 主动添加好友 |
| `Task<bool> SendLikeAsync(long botUin, long friendUin, int count = 1)` | 给好友点赞 |
| `Task<bool> SendPrivatePokeAsync(long botUin, long targetUin)` | 发送私聊戳一戳 |
| `Task<List<OneWayFriend>?> GetOneWayFriendListAsync(long botUin)` | 获取单向好友列表 |

### 群操作

| 方法签名 | 说明 |
|---|---|
| `Task<List<Group>?> GetGroupListAsync(long botUin)` | 获取群列表 |
| `Task<Group?> GetGroupInfoAsync(long botUin, long groupUin)` | 获取群信息 |
| `Task<List<GroupMember>?> GetGroupMemberListAsync(long botUin, long groupUin)` | 获取群成员列表 |
| `Task<GroupMember?> GetGroupMemberInfoAsync(long botUin, long groupUin, long memberUin)` | 获取群成员详情 |
| `Task<bool> SetGroupMemberCardAsync(long botUin, long groupUin, long memberUin, string card)` | 设置群成员名片 |
| `Task<bool> SetGroupMemberTitleAsync(long botUin, long groupUin, long memberUin, string title)` | 设置群成员专属头衔 |
| `Task<bool> SetGroupNameAsync(long botUin, long groupUin, string newName)` | 修改群名称 |
| `Task<bool> SetGroupRemarkAsync(long botUin, long groupUin, string remark)` | 设置群备注 |
| `Task<bool> SetGroupPortraitAsync(long botUin, long groupUin, string imagePath)` | 设置群头像（本地图片路径） |
| `Task LeaveGroupAsync(long botUin, long groupUin, bool isDismiss = false)` | Bot 退出群聊 |
| `Task<bool> SendGroupAddRequestAsync(long botUin, long groupUin, string? message = null)` | Bot 申请加群 |
| `Task<bool> SendGroupPokeAsync(long botUin, long groupUin, long targetUin)` | 发送群聊戳一戳 |

### 群管理

| 方法签名 | 说明 |
|---|---|
| `Task SetGroupMemberMuteAsync(long botUin, long groupUin, long memberUin, int durationSeconds)` | 设置群成员禁言；durationSeconds=0 解除禁言，单位秒 |
| `Task SetGroupWholeMuteAsync(long botUin, long groupUin, bool enable)` | 全员禁言开关 |
| `Task SetGroupAdminAsync(long botUin, long groupUin, long memberUin, bool isAdmin)` | 设置/取消管理员 |
| `Task KickGroupMemberAsync(long botUin, long groupUin, long memberUin, bool rejectAddRequest = false)` | 踢出群成员；rejectAddRequest=true 拒绝再次申请入群 |
| `Task<List<ShutUpMember>?> GetGroupShutListAsync(long botUin, long groupUin)` | 获取禁言列表 |
| `Task RecallMessageAsync(long botUin, long messageId)` | 撤回消息，支持群聊、私聊 |

### 好友/群请求处理

| 方法签名 | 说明 |
|---|---|
| `Task<List<FriendAddRequest>?> GetFriendAddRequestsAsync(long botUin)` | 获取好友请求列表 |
| `Task<List<FriendAddRequest>?> GetIgnoredFriendAddRequestsAsync(long botUin)` | 获取被过滤的好友请求 |
| `Task<bool> HandleFriendAddRequestAsync(long botUin, string flag, bool approve, string? remark = null)` | 处理单个好友请求 |
| `Task<bool> HandleIgnoredFriendAddRequestAsync(long botUin, string flag, bool approve, string? remark = null)` | 处理被过滤的好友请求 |
| `Task<List<GroupAddRequest>?> GetGroupAddRequestsAsync(long botUin, long? groupUin = null)` | 获取加群请求列表 |
| `Task<bool> HandleGroupAddRequestAsync(long botUin, string flag, bool approve, string? reason = null)` | 处理单个加群请求 |
| `Task<int> HandleGroupAddRequestsAsync(long botUin, List<GroupAddRequest> requests, bool approve, string? reason = null)` | 批量处理加群请求（返回成功数） |
| `Task<List<GroupAddRequest>?> GetIgnoredGroupAddRequestsAsync(long botUin, long? groupUin = null)` | 获取被过滤的加群请求 |
| `Task<bool> HandleIgnoredGroupAddRequestAsync(long botUin, string flag, bool approve, string? reason = null)` | 处理被过滤的加群请求 |

### 群文件

| 方法签名 | 说明 |
|---|---|
| `Task<GroupFileSystemInfo?> GetGroupFileSystemInfoAsync(long botUin, long groupUin)` | 获取群文件系统信息（空间配额） |
| `Task<List<GroupFile>?> GetGroupRootFilesAsync(long botUin, long groupUin)` | 获取根目录文件列表 |
| `Task<List<GroupFile>?> GetGroupFilesByFolderAsync(long botUin, long groupUin, string folderId)` | 获取子目录文件列表 |
| `Task<bool> UploadGroupFileAsync(long botUin, long groupUin, string filePath, string? fileName = null, string? folderId = null)` | 上传群文件，可指定目标文件夹 |
| `Task<bool> DeleteGroupFileAsync(long botUin, long groupUin, string fileId)` | 删除群文件 |
| `Task<bool> RenameGroupFileAsync(long botUin, long groupUin, string fileId, string newName)` | 重命名群文件 |
| `Task<bool> MoveGroupFileAsync(long botUin, long groupUin, string fileId, string targetFolderId = "")` | 移动群文件；targetFolderId 为空字符串移至根目录 |
| `Task<bool> TransGroupFileAsync(long botUin, long groupUin, string fileId, int busId)` | 转存临时文件为永久文件 |
| `Task<string?> GetGroupFileUrlAsync(long botUin, long groupUin, string fileId)` | 获取群文件下载链接 |
| `Task<bool> CreateGroupFileFolderAsync(long botUin, long groupUin, string folderName)` | 创建群文件夹 |
| `Task<bool> DeleteGroupFolderAsync(long botUin, long groupUin, string folderId)` | 删除群文件夹 |

### 私聊文件

| 方法签名 | 说明 |
|---|---|
| `Task<bool> UploadPrivateFileAsync(long botUin, long userId, string filePath, string? fileName = null)` | 上传本地文件到私聊 |
| `Task<PrivateFileInfo?> GetPrivateFileUrlAsync(long botUin, string fileId)` | 获取私聊文件下载链接 |

### 群公告

| 方法签名 | 说明 |
|---|---|
| `Task<List<GroupNotice>?> GetGroupNoticeListAsync(long botUin, long groupUin)` | 获取群公告列表 |
| `Task<bool> SendGroupNoticeAsync(long botUin, long groupUin, string title, string content)` | 发布群公告 |
| `Task<bool> DeleteGroupNoticeAsync(long botUin, long groupUin, string noticeId)` | 删除群公告（需要群主权限） |

### 精华消息

| 方法签名 | 说明 |
|---|---|
| `Task<bool> SetEssenceMessageAsync(long botUin, long messageId)` | 设为精华消息 |
| `Task<bool> DeleteEssenceMessageAsync(long botUin, long messageId)` | 取消精华消息 |
| `Task<List<EssenceMessage>?> GetEssenceMessageListAsync(long botUin, long groupUin)` | 获取精华消息列表 |

### 群荣誉与 @全体

| 方法签名 | 说明 |
|---|---|
| `Task<GroupHonorInfo?> GetGroupHonorInfoAsync(long botUin, long groupUin)` | 获取群荣誉信息 |
| `Task<GroupAtAllRemainInfo?> GetGroupAtAllRemainAsync(long botUin, long groupUin)` | 获取 @全体成员 剩余次数 |

### 消息历史与详情

| 方法签名 | 说明 |
|---|---|
| `Task<MessageDetail?> GetMessageAsync(long botUin, long messageId)` | 按 ID 获取单条消息详情 |
| `Task<List<MessageDetail>?> GetGroupMessageHistoryAsync(long botUin, long groupUin, int count = 20, long? messageSeq = null)` | 获取群消息历史 |
| `Task<List<MessageDetail>?> GetFriendMessageHistoryAsync(long botUin, long userId, int count = 20, long? messageSeq = null)` | 获取私聊消息历史 |

### 富媒体获取

| 方法签名 | 说明 |
|---|---|
| `Task<ImageDetail?> GetImageAsync(long botUin, string fileId)` | 获取图片消息详情（含下载链接） |
| `Task<RecordDetail?> GetRecordAsync(long botUin, string fileId)` | 获取语音消息详情 |
| `Task<VideoDetail?> GetVideoAsync(long botUin, string fileId)` | 获取视频消息详情 |

### 头像 URL

| 方法签名 | 说明 |
|---|---|
| `Task<string> GetUserAvatarUrl(long userId, int size = 640)` | 获取用户头像 URL |
| `Task<string> GetGroupAvatarUrl(long groupId, int size = 640)` | 获取群头像 URL |

### OCR 文字识别

| 方法签名 | 说明 |
|---|---|
| `Task<OcrResult?> OcrImageAsync(long botUin, string imageUrl)` | 图片 OCR（通过 imageUrl） |
| `Task<OcrResult?> OcrImageFromFileAsync(long botUin, string imagePath)` | 图片 OCR（通过本地路径） |

### 日志

| 方法签名 | 说明 |
|---|---|
| `Task<bool> WriteLog(string logContent, Color logColor = default(Color))` | 输出带颜色日志到 UniQQ 日志面板 |

**日志颜色参考**：

```csharp
Color.White  // 普通信息
Color.Gray   // 调试信息
Color.Yellow // 警告
Color.Red    // 错误
Color.Green  // 成功
```

---

## IPluginContext 属性

除了上述方法，`IPluginContext` 还暴露以下属性：

| 属性 | 类型 | 说明 | 可用阶段 |
|---|---|---|---|
| `Manifest` | `PluginManifest` | 当前插件的 manifest 信息 | OnEnable 及之后 |
| `DataPath` | `string` | 插件专属数据目录（`Data/plugins/{id}/`） | OnEnable 及之后 |
| `Events` | `IEventBus` | 事件总线实例 | OnEnable 及之后 |

---

## IEventBus — 事件总线

```csharp
public interface IEventBus
{
    // 订阅事件。TEvent 必须继承 EventBase
    void On<TEvent>(Func<TEvent, Task> handler) where TEvent : EventBase;

    // 取消订阅该插件的指定事件类型
    void Off<TEvent>() where TEvent : EventBase;
}
```

**注意**：

- `On<T>` 会自动取消该插件之前对同类型事件的订阅（同一类型只能绑定一个 handler）
- `Off<T>` 取消订阅后不会再收到该类型事件
- 同一插件的多个事件类型需分别 On / Off

```csharp
// 订阅多个事件
Context.Events.On<GroupMessageEvent>(OnGroupMessage);
Context.Events.On<PrivateMessageEvent>(OnPrivateMessage);
Context.Events.On<FriendRequestEvent>(OnFriendRequest);

// 取消订阅
Context.Events.Off<GroupMessageEvent>();
Context.Events.Off<PrivateMessageEvent>();
Context.Events.Off<FriendRequestEvent>();
```

---

## IPluginContext 完整方法速查

| # | 方法签名 |
|---|---|
| 1 | `Task<bool> ReloadThisPlugin()` |
| 2 | `Task<JObject?> Call_ApiAsync(long botUin, string action, JObject? parameters = null)` |
| 3 | `Task<bool> OnlineBotAsync(long botUin)` |
| 4 | `Task<bool> OfflineBotAsync(long botUin)` |
| 5 | `Task<bool> SetBotInfoAsync(long botUin, string? nickname = null, int? gender = null, string? birthday = null, string? location = null, string? personalNote = null)` |
| 6 | `Task<bool> SetBotAvatarAsync(long botUin, string imagePath)` |
| 7 | `Task<bool> SetBotSignatureAsync(long botUin, string longnick)` |
| 8 | `Task<bool> SetMessageEmojiLikeAsync(long botUin, long messageId, string emojiId)` |
| 9 | `Task<bool> SendGroupForwardMsgAsync(long botUin, long groupId, IEnumerable<ForwardNode>? nodes)` |
| 10 | `Task<Dictionary<string, string>?> GetCookiesAsync(long botUin, string? domain = null)` |
| 11 | `Task<string?> GetCsrfTokenAsync(long botUin)` |
| 12 | `Task<Dictionary<string, object>?> GetCredentialsAsync(long botUin, string domain = "qun.qq.com")` |
| 13 | `Task<string?> GetRKeyAsync(long botUin)` |
| 14 | `Task<string?> GetClientKeyAsync(long botUin)` |
| 15 | `Task<bool> SendPrivateForwardMsgAsync(long botUin, long userId, IEnumerable<ForwardNode>? nodes)` |
| 16 | `Task<List<Bot>?> GetBotListAsync()` |
| 17 | `Task<UniQQApplicationInfo> UniQQApplicationInfo()` |
| 18 | `Task SendGroupMessageAsync(long botUin, long groupId, Message message)` |
| 19 | `Task SendPrivateMessageAsync(long botUin, long userId, Message message)` |
| 20 | `Task SendGroupTempMessageAsync(long botUin, long groupUin, long userId, Message message)` |
| 21 | `Task<Bot?> GetLoginBotInfoAsync(long botUin)` |
| 22 | `Task<List<Friend>?> GetFriendListAsync(long botUin)` |
| 23 | `Task<List<Group>?> GetGroupListAsync(long botUin)` |
| 24 | `Task<string> GetUserAvatarUrl(long userId, int size = 640)` |
| 25 | `Task<string> GetGroupAvatarUrl(long groupId, int size = 640)` |
| 26 | `Task<List<GroupMember>?> GetGroupMemberListAsync(long botUin, long groupUin)` |
| 27 | `Task<Friend?> GetFriendInfoAsync(long botUin, long FriendUin)` |
| 28 | `Task<SDK.Models.Group?> GetGroupInfoAsync(long botUin, long groupUin)` |
| 29 | `Task<GroupMember?> GetGroupMemberInfoAsync(long botUin, long groupUin, long memberUin)` |
| 30 | `Task SetGroupMemberMuteAsync(long botUin, long groupUin, long memberUin, int durationSeconds)` |
| 31 | `Task SetGroupWholeMuteAsync(long botUin, long groupUin, bool enable)` |
| 32 | `Task SetGroupAdminAsync(long botUin, long groupUin, long memberUin, bool isAdmin)` |
| 33 | `Task RecallMessageAsync(long botUin, long messageId)` |
| 34 | `Task KickGroupMemberAsync(long botUin, long groupUin, long memberUin, bool rejectAddRequest = false)` |
| 35 | `Task LeaveGroupAsync(long botUin, long groupUin, bool isDismiss = false)` |
| 36 | `Task<List<GroupNotice>?> GetGroupNoticeListAsync(long botUin, long groupUin)` |
| 37 | `Task<bool> SendGroupNoticeAsync(long botUin, long groupUin, string title, string content)` |
| 38 | `Task<bool> DeleteGroupNoticeAsync(long botUin, long groupUin, string noticeId)` |
| 39 | `Task<bool> SetEssenceMessageAsync(long botUin, long messageId)` |
| 40 | `Task<bool> DeleteEssenceMessageAsync(long botUin, long messageId)` |
| 41 | `Task<List<EssenceMessage>?> GetEssenceMessageListAsync(long botUin, long groupUin)` |
| 42 | `Task<GroupFileSystemInfo?> GetGroupFileSystemInfoAsync(long botUin, long groupUin)` |
| 43 | `Task<List<GroupFile>?> GetGroupRootFilesAsync(long botUin, long groupUin)` |
| 44 | `Task<List<GroupFile>?> GetGroupFilesByFolderAsync(long botUin, long groupUin, string folderId)` |
| 45 | `Task<bool> UploadGroupFileAsync(long botUin, long groupUin, string filePath, string? fileName = null, string? folderId = null)` |
| 46 | `Task<bool> DeleteGroupFileAsync(long botUin, long groupUin, string fileId)` |
| 47 | `Task<bool> CreateGroupFileFolderAsync(long botUin, long groupUin, string folderName)` |
| 48 | `Task<bool> DeleteGroupFolderAsync(long botUin, long groupUin, string folderId)` |
| 49 | `Task<string?> GetGroupFileUrlAsync(long botUin, long groupUin, string fileId)` |
| 50 | `Task<List<GroupAddRequest>?> GetGroupAddRequestsAsync(long botUin, long? groupUin = null)` |
| 51 | `Task<bool> HandleGroupAddRequestAsync(long botUin, string flag, bool approve, string? reason = null)` |
| 52 | `Task<int> HandleGroupAddRequestsAsync(long botUin, List<GroupAddRequest> requests, bool approve, string? reason = null)` |
| 53 | `Task<List<FriendAddRequest>?> GetFriendAddRequestsAsync(long botUin)` |
| 54 | `Task<bool> HandleFriendAddRequestAsync(long botUin, string flag, bool approve, string? remark = null)` |
| 55 | `Task<List<FriendAddRequest>?> GetIgnoredFriendAddRequestsAsync(long botUin)` |
| 56 | `Task<bool> HandleIgnoredFriendAddRequestAsync(long botUin, string flag, bool approve, string? remark = null)` |
| 57 | `Task<bool> SetFriendRemarkAsync(long botUin, long friendUin, string remark)` |
| 58 | `Task<bool> DeleteFriendAsync(long botUin, long friendUin)` |
| 59 | `Task<List<GroupAddRequest>?> GetIgnoredGroupAddRequestsAsync(long botUin, long? groupUin = null)` |
| 60 | `Task<bool> HandleIgnoredGroupAddRequestAsync(long botUin, string flag, bool approve, string? reason = null)` |
| 61 | `Task<GroupHonorInfo?> GetGroupHonorInfoAsync(long botUin, long groupUin)` |
| 62 | `Task<GroupAtAllRemainInfo?> GetGroupAtAllRemainAsync(long botUin, long groupUin)` |
| 63 | `Task<bool> SetGroupMemberTitleAsync(long botUin, long groupUin, long memberUin, string title)` |
| 64 | `Task<List<ShutUpMember>?> GetGroupShutListAsync(long botUin, long groupUin)` |
| 65 | `Task<bool> SetGroupMemberCardAsync(long botUin, long groupUin, long memberUin, string card)` |
| 66 | `Task<bool> SetGroupNameAsync(long botUin, long groupUin, string newName)` |
| 67 | `Task<bool> SetGroupRemarkAsync(long botUin, long groupUin, string remark)` |
| 68 | `Task<bool> SetGroupPortraitAsync(long botUin, long groupUin, string imagePath)` |
| 69 | `Task<bool> SendGroupPokeAsync(long botUin, long groupUin, long targetUin)` |
| 70 | `Task<bool> SendPrivatePokeAsync(long botUin, long targetUin)` |
| 71 | `TTask<bool> SendLikeAsync(long botUin, long friendUin, int count = 1)` |
| 72 | `Task<List<OneWayFriend>?> GetOneWayFriendListAsync(long botUin)` |
| 73 | `Task<bool> RenameGroupFileAsync(long botUin, long groupUin, string fileId, string newName)` |
| 74 | `Task<bool> TransGroupFileAsync(long botUin, long groupUin, string fileId, int busId)` |
| 75 | `Task<bool> UploadPrivateFileAsync(long botUin, long userId, string filePath, string? fileName = null)` |
| 76 | `Task<PrivateFileInfo?> GetPrivateFileUrlAsync(long botUin, string fileId)` |
| 77 | `Task<bool> SendGroupAddRequestAsync(long botUin, long groupUin, string? message = null)` |
| 78 | `Task<bool> SendFriendAddRequestAsync(long botUin, long targetUin, string? message = null)` |
| 79 | `Task<MessageDetail?> GetMessageAsync(long botUin, long messageId)` |
| 80 | `Task<List<MessageDetail>?> GetGroupMessageHistoryAsync(long botUin, long groupUin, int count = 20, long? messageSeq = null)` |
| 81 | `Task<List<MessageDetail>?> GetFriendMessageHistoryAsync(long botUin, long userId, int count = 20, long? messageSeq = null)` |
| 82 | `Task<ImageDetail?> GetImageAsync(long botUin, string fileId)` |
| 83 | `Task<RecordDetail?> GetRecordAsync(long botUin, string fileId)` |
| 84 | `Task<VideoDetail?> GetVideoAsync(long botUin, string fileId)` |
| 85 | `Task<bool> MoveGroupFileAsync(long botUin, long groupUin, string fileId, string targetFolderId = "")` |
| 86 | `Task<OcrResult?> OcrImageAsync(long botUin, string imageUrl)` |
| 87 | `Task<OcrResult?> OcrImageFromFileAsync(long botUin, string imagePath)` |
| 88 | `Task<bool> WriteLog(string logContent, Color logColor = default)` |
| 88 | `Task<long[]> SendGroupMsgReturnIdAsync(long botUin, long groupId, Message message)` |
| 88 | `Task<long> SendPrivateMsgReturnIdAsync(long botUin, long userId, Message message)` |
| 88 | `Task<long> SendGroupTempMsgReturnIdAsync(long botUin, long groupUin, long userId, Message message)` |

---

## EventBase — 事件基类

所有事件都继承自 `EventBase`：

```csharp
public class EventBase
{
    public DateTime Time { get; set; }             // 事件发生时间
    public IPluginContext Context { get; set; }    // 插件上下文（由框架自动注入）
}
```

---

## 枚举

### MemberRole — 成员角色

```csharp
public enum MemberRole
{
    Member,  // 普通成员
    Admin,   // 管理员
    Owner    // 群主
}
```

### MessageType — 消息类型

```csharp
public enum MessageType
{
    Group,      // 群消息
    Private,    // 私聊消息
    Temp        // 临时会话
}
```

### BotStatus — 机器人状态

```csharp
public enum BotStatus
{
    Offline,
    Online,
    Connecting,
    Disconnecting
}
```

### OnlineStatus — 在线状态

```csharp
public enum OnlineStatus
{
    Online,
    Offline,
    Busy,
    Away,
    Hidden
}
```

---

## 下一步

- [事件系统](/concepts/events) — 所有事件类型的属性和用法
- [消息构建器](/api/utils) — `MessageBuilder` 和 `Message` 详细用法
- [示例教程](/examples/basic) — 完整可运行示例


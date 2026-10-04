# 第三方机器人与腾讯官方机器人接口差异表

本文档用于帮助插件开发者判断：哪些 UniQQ SDK 接口可以跨底层通用，哪些只适合第三方机器人，哪些是腾讯官方机器人专属能力。

当前 UniQQ 支持第三方机器人与腾讯官方机器人并存。插件开发时建议遵循下面的顺序：

1. **能用通用接口就用通用接口**，例如普通文本消息、机器人列表、已知群列表、日志等。
2. **需要 QQ 客户端原生能力时，按第三方机器人能力设计**，例如群管理、群文件、好友管理、合并转发、OCR 等。
3. **需要官方机器人独有能力时，使用 `Context.Official`**，例如 Markdown、Ark、Embed、按钮、官方富媒体上传、官方 OpenAPI 透传。

---

## 状态说明

| 状态 | 说明 |
|---|---|
| 完整支持 | 当前底层可以按接口语义工作 |
| 受限支持 | 接口能调用，但数据来源、发送内容或行为受底层限制 |
| 不支持 | 底层没有等价能力，UniQQ 会写日志并返回失败值 |
| 专属接口 | 只属于某一类底层，不建议另一类底层调用 |

腾讯官方机器人调用不支持的通用接口时，一般不会让插件崩溃，而是由 UniQQ 统一写系统日志，并返回 `false`、`null`、`0`、空数组或空列表。

---

## 通用接口

这些接口第三方机器人和腾讯官方机器人都可以调用。官方机器人的部分接口为受限支持。

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 | 说明 |
|---|---|---|---|
| `OnlineBotAsync` | 完整支持 | 完整支持 | 根据机器人类型进入对应上线流程 |
| `OfflineBotAsync` | 完整支持 | 完整支持 | 根据机器人类型进入对应下线流程 |
| `GetBotListAsync` | 完整支持 | 完整支持 | 返回 UniQQ 已配置机器人及运行状态 |
| `GetLoginBotInfoAsync` | 完整支持 | 受限支持 | 官方返回 UniQQ 已知的官方机器人信息 |
| `UniQQApplicationInfo` | 完整支持 | 完整支持 | 与底层无关 |
| `WriteLog` | 完整支持 | 完整支持 | 与底层无关 |
| `ReloadThisPlugin` | 完整支持 | 完整支持 | 与底层无关 |
| `GetUserAvatarUrl` | 完整支持 | 受限支持 | 官方虚拟用户 ID 不一定等价于真实 QQ 号 |
| `GetGroupAvatarUrl` | 完整支持 | 受限支持 | 官方虚拟群 ID 不一定等价于真实群号 |

---

## 通用消息接口

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 | 说明 |
|---|---|---|---|
| `SendGroupMsgReturnIdAsync` | 完整支持 | 受限支持 | 官方支持文本、图片 URL、部分 @；本地图片、XML、JSON 等会降级 |
| `SendPrivateMsgReturnIdAsync` | 完整支持 | 受限支持 | 官方支持文本、图片 URL |
| `SendGroupMessageAsync` | 完整支持 | 受限支持 | 旧版兼容接口，建议改用返回消息 ID 的版本 |
| `SendPrivateMessageAsync` | 完整支持 | 受限支持 | 旧版兼容接口，建议改用返回消息 ID 的版本 |

消息段差异：

| 消息段 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| 文本 `TextSegment` | 完整支持 | 完整支持 |
| @成员 `AtSegment` | 完整支持 | 已知映射成员支持；未找到官方 `openid` 时降级为文本 |
| @全体 `AtSegment.Target == 0` | 完整支持 | 降级为文本 `@全体成员` |
| 图片 `ImageSegment` | 支持本地、URL、file_id 等 | 支持 http/https 图片 URL；本地文件暂不直接发送 |
| 回复 `ReplySegment` | 支持 | 通用发送中忽略；官方原生接口可传 `replyMessageId` |
| 表情 `FaceSegment` | 支持 | 降级为文本 `[表情:id]` |
| 语音 / 视频 / 文件 | 支持 | 通用发送中降级；官方专属富媒体请使用 `Context.Official` |
| JSON / XML | 支持 | 降级为“不支持”文本 |
| 音乐 / 分享 | 支持 | 降级为普通类型标记文本 |
| 合并转发 | 支持 | 无等价能力 |

---

## 通用但受限的查询接口

这些接口在官方机器人中不会返回完整 QQ 客户端数据，只返回 UniQQ 已经通过事件收集到的已知对象。

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 | 官方限制 |
|---|---|---|---|
| `GetGroupListAsync` | 返回完整群列表 | 返回已知群缓存 | 官方平台不提供完整传统 QQ 群列表 |
| `GetGroupInfoAsync` | 返回真实群资料 | 返回已知群或虚拟群信息 | 数据来自事件映射 |
| `GetFriendListAsync` | 返回好友列表 | 返回已知单聊用户缓存 | 官方平台不提供完整 QQ 好友列表 |
| `GetFriendInfoAsync` | 返回好友资料 | 返回已知用户信息 | 数据来自事件映射 |
| `GetGroupMemberListAsync` | 返回群成员列表 | 返回已知群成员缓存 | 官方平台不提供完整群成员列表 |
| `GetGroupMemberInfoAsync` | 返回群成员资料 | 返回已知群成员信息 | 数据来自事件映射 |

官方机器人里看到的 `User_Id`、`Group_Id` 是 UniQQ 生成的虚拟数字 ID。插件不要把它们当作真实 QQ 号或真实群号。

---

## 第三方机器人专用接口

以下接口依赖 NapCat / QQ 客户端能力。腾讯官方机器人当前没有等价能力，调用时会写日志并返回失败值。

### 账号资料与客户端凭证

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `SetBotInfoAsync` | 完整支持 | 不支持 |
| `SetBotAvatarAsync` | 完整支持 | 不支持 |
| `SetBotSignatureAsync` | 完整支持 | 不支持 |
| `GetCookiesAsync` | 完整支持 | 不支持 |
| `GetCsrfTokenAsync` | 完整支持 | 不支持 |
| `GetCredentialsAsync` | 完整支持 | 不支持 |
| `GetRKeyAsync` | 完整支持 | 不支持 |
| `GetClientKeyAsync` | 完整支持 | 不支持 |

### 传统消息能力

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `Call_ApiAsync` 调用 OneBot/NapCat action | 完整支持 | 不支持 OneBot action；官方路径请用 `Context.Official.CallOfficialOpenApiAsync` |
| `SendGroupTempMsgReturnIdAsync` | 完整支持 | 不支持 |
| `SendGroupTempMessageAsync` | 完整支持 | 不支持 |
| `SendGroupForwardMsgAsync` | 完整支持 | 不支持 |
| `SendPrivateForwardMsgAsync` | 完整支持 | 不支持 |
| `GetForwardMsgAsync` | 完整支持 | 不支持 |
| `GetMessageAsync` | 完整支持 | 不支持 |
| `GetGroupMessageHistoryAsync` | 完整支持 | 不支持 |
| `GetFriendMessageHistoryAsync` | 完整支持 | 不支持 |
| `SetMessageEmojiLikeAsync` | 完整支持 | 不支持 |
| `GetImageAsync` | 完整支持 | 不支持 |
| `GetRecordAsync` | 完整支持 | 不支持 |
| `GetVideoAsync` | 完整支持 | 不支持 |
| `OcrImageAsync` | 完整支持 | 不支持 |
| `OcrImageFromFileAsync` | 完整支持 | 不支持 |

### 好友管理

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `SetFriendRemarkAsync` | 完整支持 | 不支持 |
| `DeleteFriendAsync` | 完整支持 | 不支持 |
| `SendFriendAddRequestAsync` | 完整支持 | 不支持 |
| `GetFriendAddRequestsAsync` | 完整支持 | 不支持 |
| `HandleFriendAddRequestAsync` | 完整支持 | 不支持 |
| `GetIgnoredFriendAddRequestsAsync` | 完整支持 | 不支持 |
| `HandleIgnoredFriendAddRequestAsync` | 完整支持 | 不支持 |
| `SendLikeAsync` | 完整支持 | 不支持 |
| `SendPrivatePokeAsync` | 完整支持 | 不支持 |
| `GetOneWayFriendListAsync` | 完整支持 | 不支持 |

### 群管理

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `SetGroupMemberMuteAsync` | 完整支持 | 不支持 |
| `SetGroupWholeMuteAsync` | 完整支持 | 不支持 |
| `SetGroupAdminAsync` | 完整支持 | 不支持 |
| `RecallMessageAsync` | 完整支持 | 不支持 |
| `KickGroupMemberAsync` | 完整支持 | 不支持 |
| `LeaveGroupAsync` | 完整支持 | 不支持 |
| `SetGroupMemberTitleAsync` | 完整支持 | 不支持 |
| `SetGroupMemberCardAsync` | 完整支持 | 不支持 |
| `SetGroupNameAsync` | 完整支持 | 不支持 |
| `SetGroupRemarkAsync` | 完整支持 | 不支持 |
| `SetGroupPortraitAsync` | 完整支持 | 不支持 |
| `SendGroupPokeAsync` | 完整支持 | 不支持 |
| `GetGroupShutListAsync` | 完整支持 | 不支持 |
| `GetGroupHonorInfoAsync` | 完整支持 | 不支持 |
| `GetGroupAtAllRemainAsync` | 完整支持 | 不支持 |

### 入群与邀请请求

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `GetGroupAddRequestsAsync` | 完整支持 | 不支持 |
| `HandleGroupAddRequestAsync` | 完整支持 | 不支持 |
| `HandleGroupAddRequestsAsync` | 完整支持 | 不支持 |
| `GetIgnoredGroupAddRequestsAsync` | 完整支持 | 不支持 |
| `HandleIgnoredGroupAddRequestAsync` | 完整支持 | 不支持 |
| `SendGroupAddRequestAsync` | 完整支持 | 不支持 |

### 群文件、私聊文件、公告、精华

| SDK 接口 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `GetGroupFileSystemInfoAsync` | 完整支持 | 不支持 |
| `GetGroupRootFilesAsync` | 完整支持 | 不支持 |
| `GetGroupFilesByFolderAsync` | 完整支持 | 不支持 |
| `UploadGroupFileAsync` | 完整支持 | 不支持 |
| `DeleteGroupFileAsync` | 完整支持 | 不支持 |
| `CreateGroupFileFolderAsync` | 完整支持 | 不支持 |
| `DeleteGroupFolderAsync` | 完整支持 | 不支持 |
| `GetGroupFileUrlAsync` | 完整支持 | 不支持 |
| `RenameGroupFileAsync` | 完整支持 | 不支持 |
| `MoveGroupFileAsync` | 完整支持 | 不支持 |
| `TransGroupFileAsync` | 完整支持 | 不支持 |
| `UploadPrivateFileAsync` | 完整支持 | 不支持 |
| `GetPrivateFileUrlAsync` | 完整支持 | 不支持 |
| `GetGroupNoticeListAsync` | 完整支持 | 不支持 |
| `SendGroupNoticeAsync` | 完整支持 | 不支持 |
| `DeleteGroupNoticeAsync` | 完整支持 | 不支持 |
| `SetEssenceMessageAsync` | 完整支持 | 不支持 |
| `DeleteEssenceMessageAsync` | 完整支持 | 不支持 |
| `GetEssenceMessageListAsync` | 完整支持 | 不支持 |

---

## 腾讯官方机器人专用接口

以下接口只服务腾讯官方机器人。第三方机器人插件不需要调用这些接口。

| 官方专属接口 | 说明 |
|---|---|
| `Context.Official.IsOfficialBotAsync` | 判断机器人是否为官方机器人 |
| `Context.Official.SendOfficialGroupMarkdownMessageAsync` | 发送官方群 Markdown 消息 |
| `Context.Official.SendOfficialPrivateMarkdownMessageAsync` | 发送官方单聊 Markdown 消息 |
| `Context.Official.SendOfficialGroupArkMessageAsync` | 发送官方群 Ark 消息 |
| `Context.Official.SendOfficialPrivateArkMessageAsync` | 发送官方单聊 Ark 消息 |
| `Context.Official.SendOfficialGroupEmbedMessageAsync` | 发送官方群 Embed 消息 |
| `Context.Official.SendOfficialPrivateEmbedMessageAsync` | 发送官方单聊 Embed 消息 |
| `Context.Official.UploadOfficialGroupMediaByUrlAsync` | 上传官方群富媒体资源 |
| `Context.Official.UploadOfficialPrivateMediaByUrlAsync` | 上传官方单聊富媒体资源 |
| `Context.Official.SendOfficialGroupMediaMessageAsync` | 发送官方群富媒体消息 |
| `Context.Official.SendOfficialPrivateMediaMessageAsync` | 发送官方单聊富媒体消息 |
| `Context.Official.AcknowledgeOfficialInteractionAsync` | ACK 官方按钮交互事件 |
| `Context.Official.GetOfficialIdMappingAsync` | 查询 UniQQ 虚拟 ID 与官方 ID 映射 |
| `Context.Official.CallOfficialOpenApiAsync` | 透传调用腾讯官方 OpenAPI |

---

## 事件差异

### 通用事件

| UniQQ 事件 | 第三方机器人 | 腾讯官方机器人 |
|---|---|---|
| `BotOnlineEvent` | 支持 | 支持 |
| `BotOfflineEvent` | 支持 | 当前主要由 UniQQ Session/Gateway 状态管理，不保证以 SDK 事件形式分发 |
| `HeartbeatEvent` | 支持 | 当前不作为 SDK 心跳事件分发；官方 Gateway 心跳由 UniQQ 内部维护 |
| `GroupMessageEvent` | 支持 | 支持，由官方群消息事件转换 |
| `PrivateMessageEvent` | 支持 | 支持，由官方 C2C 消息事件转换 |
| `BotJoinGroupEvent` | 支持 | 支持，由 `GROUP_ADD_ROBOT` 转换 |
| `BotKickEvent` | 支持 | 支持，由 `GROUP_DEL_ROBOT` 转换 |
| `FriendAddedEvent` | 支持 | 支持，由 `FRIEND_ADD` 转换 |
| `FriendDeletedEvent` | 支持 | 支持，由 `FRIEND_DEL` 转换 |

### 第三方机器人更完整的事件

| 事件类型 | 腾讯官方机器人状态 |
|---|---|
| 群成员进群、退群、被踢、禁言、解除禁言、管理员变化 | 当前不提供等价通用转换 |
| 群消息撤回、好友消息撤回、精华消息、戳一戳、群名变更、表情回应、群文件上传 | 当前不提供等价通用转换 |
| 好友申请、加群申请、机器人被邀请加群 | 当前不提供传统 QQ 请求处理等价能力 |

### 官方机器人专属事件

| UniQQ 事件 | 说明 |
|---|---|
| `InteractionCreateEvent` | 官方按钮、指令等交互事件 |
| `MessageAuditEvent` | 官方消息审核通过或拒绝事件 |
| `ActiveMessageSettingChangedEvent` | 官方主动消息接收设置变化事件 |

---

## 开发建议

### 希望插件跨底层通用

优先使用：

- `GetBotListAsync`
- `GetLoginBotInfoAsync`
- `GetGroupListAsync`
- `GetFriendListAsync`
- `GetGroupMemberListAsync`
- `SendGroupMsgReturnIdAsync`
- `SendPrivateMsgReturnIdAsync`
- `WriteLog`
- 通用消息事件：`GroupMessageEvent`、`PrivateMessageEvent`

同时需要接受官方机器人中的数据是“已知缓存”，不是完整 QQ 客户端数据。

### 希望插件发挥第三方机器人完整能力

可以使用群管理、好友管理、群文件、公告、精华、合并转发、OCR、历史消息等接口，但需要在插件说明里标注“第三方机器人专用”。

### 希望插件发挥官方机器人特色

可以使用：

- Markdown / Ark / Embed 消息
- 官方按钮
- 官方富媒体上传
- 官方交互 ACK
- 官方 OpenAPI 透传

这类插件应在调用前使用 `Context.Official.IsOfficialBotAsync(botUin)` 判断机器人类型。

---

## 一句话总结

UniQQ 的兼容策略是：**通用能力尽量保持同一套 SDK；官方不具备的 QQ 客户端能力明确降级并写日志；官方独有能力单独放到 `Context.Official`，不破坏原有插件。**

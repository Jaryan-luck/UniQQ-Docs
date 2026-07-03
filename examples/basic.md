# 基础插件示例

从最简单的无状态插件开始，逐步展示各种功能。

---

## 示例一：复读机

最简单的插件，收到消息后原样回复。

```csharp
using UniQQ.SDK.Events.MessageEvents;
using UniQQ.SDK.Plugins;

namespace Repeater;

public class RepeaterPlugin : PluginBase
{
    public override string Name => "复读机";
    public override string Version => "1.0.0";

    public override Task OnEnable()
    {
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        Context.Events.On<PrivateMessageEvent>(OnPrivateMessage);
        return Task.CompletedTask;
    }

    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        // 原样回复群消息
        await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id, e.Message);
    }

    private async Task OnPrivateMessage(PrivateMessageEvent e)
    {
        // 原样回复私聊消息
        await Context.SendPrivateMessageAsync(e.Bot_Id, e.User_Id, e.Message);
    }

    public override Task OnDisable()
    {
        Context.Events.Off<GroupMessageEvent>();
        Context.Events.Off<PrivateMessageEvent>();
        return Task.CompletedTask;
    }

    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```
---

## 示例二：关键字回复机器人

根据特定关键字回复不同内容。
```csharp
using UniQQ.SDK.Events.MessageEvents;
using UniQQ.SDK.Plugins;

namespace KeywordBot;

public class KeywordBotPlugin : PluginBase
{
    public override string Name => "关键字回复";
    public override string Version => "1.0.0";
    
    private readonly Dictionary<string, string> _keywords = new()
    {
        { "你好", "你好呀！欢迎来到本群~" },
        { "帮助", "可用命令：\n1. 输入\"你好\"获取问候\n2. 输入\"天气\"获取天气信息\n3. 输入\"关于\"了解机器人" },
        { "天气", "今天的天气不错！适合出门散步~" },
        { "关于", "我是 UniQQ 机器人插件！" },
    };

    public override Task OnEnable()
    {
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        return Task.CompletedTask;
    }

    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        var text = e.Message.RawText.Trim();
        
        foreach (var (keyword, reply) in _keywords)
        {
            if (text.Contains(keyword))
            {
                await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id,
                    MessageBuilder.Text(reply));
                return;
            }
        }
    }

    public override Task OnDisable()
    {
        Context.Events.Off<GroupMessageEvent>();
        return Task.CompletedTask;
    }

    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```
---

## 示例三：入群欢迎

检测新成员入群并发送欢迎消息。

```csharp
using UniQQ.SDK.Builders;
using UniQQ.SDK.Events.Notice;
using UniQQ.SDK.Models;
using UniQQ.SDK.Models.Segments;
using UniQQ.SDK.Plugins;

namespace WelcomeBot;

public class WelcomePlugin : PluginBase
{
    public override string Name => "入群欢迎";
    public override string Version => "1.0.0";

    public override Task OnEnable()
    {
        Context.Events.On<MemberJoinEvent>(OnMemberJoin);
        return Task.CompletedTask;
    }

    private async Task OnMemberJoin(MemberJoinEvent e)
    {
        // 构建欢迎消息（@新成员 + 文字）
        var msg = new Message
        {
            Segments = new List<MessageSegment>
            {
                MessageBuilder.At(e.User_Id),
                MessageBuilder.Text($" 欢迎加入本群！你是本群第 {await GetMemberCount(e.Bot_Id, e.Group_Id)} 位成员~"),
                MessageBuilder.Face(76), // 添加[爱心]表情
            }
        };

        await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id, msg);
    }

    private async Task<int> GetMemberCount(long botUin, long groupUin)
    {
        var group = await Context.GetGroupInfoAsync(botUin, groupUin);
        return group.MemberCount;
    }

    public override Task OnDisable()
    {
        Context.Events.Off<MemberJoinEvent>();
        return Task.CompletedTask;
    }

    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```
---

## 示例四：点歌功能(抄Lucent的，没测试)

通过网易云音乐 API 搜索歌曲并发送音乐卡片。
```csharp
using System.Text.Json.Nodes;
using System.Text.RegularExpressions;
using UniQQ.SDK.Builders;
using UniQQ.SDK.Events.MessageEvents;
using UniQQ.SDK.Plugins;

namespace MusicBot;

public class MusicBotPlugin : PluginBase
{
    public override string Name => "点歌机器人";
    public override string Version => "1.0.0";

    public override Task OnEnable()
    {
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        return Task.CompletedTask;
    }

    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        var rawText = e.Message.RawText.TrimStart();//加去除收尾空格的写法
        Regex _whiteSpaceRegex = new Regex(@"[\s\r\n]", RegexOptions.Compiled);
        int MaxSongNameLength = 30;
        if (rawText.Length >= 2 && rawText.Substring(0, 2) == "点歌")
        {
            try
            {
                string rawSong = rawText.Substring(2);
                if (rawSong.Length > MaxSongNameLength)
                    rawSong = rawSong.Substring(0, MaxSongNameLength);

                string pureName = _whiteSpaceRegex.Replace(rawSong, "");
                if (string.IsNullOrWhiteSpace(pureName))
                {
                    await e.ReplyAsync("请输入有效的歌曲名称");
                    return;
                }
                var songId = await SearchSongIdAsync(pureName);
                if (string.IsNullOrEmpty(songId))
                {
                    await e.ReplyAsync("未找到该歌曲，请更换关键词重试");
                    return;
                }

                await e.ReplyAsync(MessageBuilder.NeteaseMusic(songId));
            }
            catch (Exception ex)
            {
                await e.ReplyAsync($"查询歌曲服务暂时出错，请稍后重试{ex}");
            }
        }
        return;

    }

    private async Task<string?> SearchSongIdAsync(string songName)
    {
        try
        {
            string url =
                $"https://music-api.gdstudio.xyz/api.php?types=search&source=netease&name={Uri.EscapeDataString(songName)}";

            using var http = new HttpClient();

            string json = await http.GetStringAsync(url);

            var songs = JsonNode.Parse(json)?.AsArray();

            if (songs == null || songs.Count == 0)
                return null;

            foreach (var song in songs)
            {
                if (song?["name"]?.ToString() == songName)
                {
                    return song["id"]?.ToString();
                }
            }

            return songs[0]?["id"]?.ToString();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex);
            return null;
        }
    }

    public override Task OnDisable()
    {
        Context.Events.Off<GroupMessageEvent>();
        return Task.CompletedTask;
    }

    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```
---

## 示例五：非常之简陋的群管助手

管理员命令：禁言、踢人、查看成员信息。
```csharp
using UniQQ.SDK.Builders;
using UniQQ.SDK.Enums;
using UniQQ.SDK.Events.MessageEvents;
using UniQQ.SDK.Models;
using UniQQ.SDK.Models.Segments;
using UniQQ.SDK.Plugins;

namespace AdminBot;

public class AdminPlugin : PluginBase
{
    public override string Name => "群管助手";
    public override string Version => "1.0.0";

    public override Task OnEnable()
    {
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        return Task.CompletedTask;
    }

    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        // 仅管理员/群主可操作
        if (e.User_Role == MemberRole.Member) return;

        var text = e.Message.RawText.Trim();
        var parts = text.Split(' ', StringSplitOptions.RemoveEmptyEntries);

        if (parts.Length < 2) return;

        var cmd = parts[0].ToLower();

        // /禁言 @某人 10（禁言10分钟）
        if (cmd == "/禁言" && parts.Length >= 3 && int.TryParse(parts[2], out var minutes))
        {
            var target = ExtractAtTarget(e.Message);
            if (target.HasValue)
            {
                await Context.SetGroupMemberMuteAsync(e.Bot_Id, e.Group_Id, target.Value, (int)(minutes * 60));
                await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id,
                    MessageBuilder.Text($"已禁言 {target.Value} {minutes} 分钟"));
            }
        }
        // /踢人 @某人
        else if (cmd == "/踢人")
        {
            var target = ExtractAtTarget(e.Message);
            if (target.HasValue)
            {
                await Context.KickGroupMemberAsync(e.Bot_Id, e.Group_Id, target.Value, false);
            }
        }
        // /群信息
        else if (cmd == "/群信息")
        {
            var group = await Context.GetGroupInfoAsync(e.Bot_Id, e.Group_Id);
            var info = $"群名称：{group.GroupName}\n群号：{group.Uin}\n成员数：{group.MemberCount}/{group.MaxMemberCount}";
            await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id,
                MessageBuilder.Text(info));
        }
    }

    /// <summary>
    /// 从消息段中提取被 @ 的 QQ 号
    /// </summary>
    private static long? ExtractAtTarget(Message message)
    {
        foreach (var seg in message.Segments)
        {
            if (seg is AtSegment at)
                return at.Target;
        }
        return null;
    }

    public override Task OnDisable()
    {
        Context.Events.Off<GroupMessageEvent>();
        return Task.CompletedTask;
    }

    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```
---

## 示例六：好友申请自动处理

自动同意好友申请并发送欢迎消息。
```csharp
using UniQQ.SDK.Builders;
using UniQQ.SDK.Events.Request;
using UniQQ.SDK.Plugins;

namespace AutoFriend;

public class AutoFriendPlugin : PluginBase
{
    public override string Name => "自动加友";
    public override string Version => "1.0.0";

    public override Task OnEnable()
    {
        Context.Events.On<FriendRequestEvent>(OnFriendRequest);
        Context.Events.On<BotGroupInviteEvent>(OnBotGroupInvite);
        return Task.CompletedTask;
    }

    private async Task OnFriendRequest(FriendRequestEvent e)
    {
        // 自动同意好友申请
        bool success = await Context.HandleFriendAddRequestAsync(
            e.Bot_Id, e.Flag, true, "");

        if (success)
        {
            // 发送欢迎私聊
            await Context.SendPrivateMessageAsync(e.Bot_Id, e.User_Id,
                MessageBuilder.Text($"你好！我是机器人 {e.Bot_Id}，已自动通过你的好友申请！"));
        }
    }

    private async Task OnBotGroupInvite(BotGroupInviteEvent e)
    {
        // 自动同意加群邀请
        bool success = await Context.HandleGroupAddRequestAsync(
            e.Bot_Id, e.Flag, true, "已自动同意");
    }

    public override Task OnDisable()
    {
        Context.Events.Off<FriendRequestEvent>();
        Context.Events.Off<BotGroupInviteEvent>();
        return Task.CompletedTask;
    }

    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```
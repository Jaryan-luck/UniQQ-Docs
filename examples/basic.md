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

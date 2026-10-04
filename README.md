# UniQQ 插件开发文档(CSharp)

> 基于 `UniQQ.SDK v1.0.9`

---
## 本文档地址 http://jaryan.work/UniQQ-Docs/


## 快速入门
> 你可以从群文件中获取这个示例插件`HelloUniQQ(含源码).zip`
```csharp
using UniQQ.SDK.Builders;
using UniQQ.SDK.Events.MessageEvents;
using UniQQ.SDK.Plugins;
namespace HelloUniQQ;
public class HelloUniQQ : PluginBase
{
    public override string Name => "我的插件";
    public override string Version => "1.0.0";

    public override Task Load() => Task.CompletedTask;
    public override Task OnEnable()
    {
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        return Task.CompletedTask;
    }
    public override Task OnSettings() => Task.CompletedTask;
    public override Task OnDisable()
    {
        Context.Events.Off<GroupMessageEvent>();
        return Task.CompletedTask;
    }
    public override Task Unload() => Task.CompletedTask;
    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        if (e.Message.RawText == "你好")
            await Context.SendGroupMsgReturnIdAsync(e.Bot_Id, e.Group_Id,
                MessageBuilder.Text("Hello! UniQQ!"));
    }
}
```

---

## 文档导航

| 章节 | 说明 |
|------|------|
| [快速上手](/guide/quickstart) | 从零开始创建你的第一个插件 |
| [环境配置](/guide/setup) | 开发环境安装与项目配置 |
| [插件生命周期](/concepts/lifecycle) | Load → OnEnable → OnDisable → Unload |
| [插件发布](/concepts/manifest) | plugin.json 配置详解 |
| [事件系统](/concepts/events) | 消息、通知、请求事件全览 |
| [API](/api/core) | PluginBase、IPluginContext、IEventBus |
| [官方机器人 API](/api/official) | 腾讯官方机器人专属 Markdown、Ark、Embed、富媒体与 OpenAPI 能力 |
| [底层能力差异表](/api/bot-compatibility) | 第三方机器人与腾讯官方机器人的接口差异 |
| [示例教程](/examples/basic) | 从基础到高级的完整示例 |

---

## 环境要求

- **.NET SDK** `10.0+`
- **Visual Studio IDE** （推荐） 或 **JetBrains Rider**
- **UniQQ 客户端**

---

> **新手提示**：如果你从未接触过 UniQQ 插件开发，请从 [快速上手](/guide/quickstart) 开始阅读。

# 快速上手

本教程将带你从零开始创建你的第一个 UniQQ 插件。

---

## 第一步：安装开发环境

确保已安装以下工具：

- **.NET 10.0 SDK**：[下载地址](https://dotnet.microsoft.com/download)
- **Visual Studio IDE** 或 **JetBrains Rider**（支持 .NET 项目即可）

验证安装：

```bash
dotnet --version
# 应输出 10.0.xxx 或更高版本
```

---

## 第二步：获取插件模板

> 从群获取文件 `UniQQ插件开发包\空插件模板(Csharp).zip`解压

模板包含以下文件：

| 文件 | 说明 |
|------|------|
| `Plugin.cs` | 纯净骨架 |
| `plugin.json` | 插件发布配置文件 |
| `ProjectName.csproj` | 项目文件 |
| `UniQQ.SDK.dll` | UniQQSDK |
| `EmptyProject.slnx` | 解决方案(Visual Studio IDE) |

---

## 第三步：创建项目

### 方式一：使用模板项目

1. 复制 `空插件模板(Csharp)` 文件夹，重命名为你的插件名称
2. 用 IDE 打开 `.csproj` 或 `.sln` 文件
3. 重命名ProjectName

### 方式二：手动创建

```bash
# 创建类库项目
dotnet new classlib -n MyUniQQPlugin -f net10.0
cd MyUniQQPlugin
```

然后修改 `.csproj` 文件：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="UniQQ.SDK">
      <HintPath>..\path\to\UniQQ.SDK.dll(你的SDK目录)</HintPath>
    </Reference>
  </ItemGroup>
</Project>
```

---

## 第四步：编写你的第一个插件

创建 `HelloWorldPlugin.cs`：

```csharp
using UniQQ.SDK;
using UniQQ.SDK.Events.MessageEvents;
using UniQQ.SDK.Plugins;

namespace MyFirstPlugin;

public class HelloWorldPlugin : PluginBase
{
    public override string Name => "HelloWorld";
    public override string Version => "1.0.0";

    public override Task OnEnable()
    {
        // 订阅群消息事件
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        return Task.CompletedTask;
    }

    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        // 收到 "hello" 时回复
        if (e.Message.RawText.Trim() == "hello")
        {
            var reply = MessageBuilder.Text("Hello! UniQQ 插件已响应！");
            await Context.SendGroupMessageAsync(e.Bot_Id, e.Group_Id, reply);
        }
    }

    public override Task OnDisable()
    {
        // 取消订阅所有事件
        Context.Events.Off<GroupMessageEvent>();
        return Task.CompletedTask;
    }

    // 以下方法无需实现，保持默认即可
    public override Task Load() => Task.CompletedTask;
    public override Task OnSettings() => Task.CompletedTask;
    public override Task Unload() => Task.CompletedTask;
}
```

---

## 第五步：创建 plugin.json

在项目根目录创建 `plugin.json`：

```json
{
  "id": "HelloWorldPlugin",
  "name": "HelloWorld",
  "author": "你的名字",
  "version": "1.0.0",
  "description": "我的第一个 UniQQ 插件",
  "runtime": "dotnet",
  "entry": "MyFirstPlugin.dll",
  "sdkVersion": "1.0.0",
  "MinFrameworkVersion": "1.0.0",
  "architecture": "any",
  "website": ""
}
```

---

## 第六步：编译插件

```bash
dotnet build -c Release
```

编译成功后，在 `bin/Release/net10.0/` 目录下会生成你的插件 DLL。

---

## 第七步：打包插件

1. 将你的插件 DLL 与 plugin.json 一起添加到压缩包(rar格式)
2. 将压缩包的后缀名改为uniqq

---


## 第八步：安装到 UniQQ

1. 打开 UniQQ 客户端
2. 进入 **插件**
3. 点击 **安装新插件**，选择你打包的插件
4. 启用插件，在群里发送 `hello` 测试效果

---

## 下一步

- 了解 [插件生命周期](/concepts/lifecycle) 以掌握更复杂的初始化逻辑
- 学习 [事件系统](/concepts/events) 来响应更多类型的消息
- 查看 [API 参考](/api/core) 了解完整的 SDK 能力

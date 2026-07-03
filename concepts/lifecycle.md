# 插件生命周期

UniQQ 插件有严格的生命周期顺序，理解它是开发稳定插件的基础。

---

## 生命周期总览

```
  加载插件
     │
     ▼
┌──────────┐
│   Load   │  ← 插件被框架载入时调用（最先执行，仅一次）
└────┬─────┘
     │
     ▼
┌──────────┐
│ OnEnable │  ← 插件被启用时调用（订阅事件、初始化资源）
└────┬─────┘
     │
     ▼
  运行事件  ← 处理消息、通知、请求等
     │
     ▼
┌──────────┐
│OnDisable │  ← 插件被禁用时调用（取消事件订阅、释放资源）
└────┬─────┘
     │
     ▼
┌──────────┐
│  Unload  │  ← 插件被卸载时调用（清理收尾）
└──────────┘
```

---

## 各阶段详解

### Load() — 加载阶段

**执行时机**：插件 DLL 被框架载入时，仅执行一次。

**适用场景**：
- 全局静态资源加载
- 读取配置文件
- 初始化不依赖 `Context` 的变量

**注意事项**：
- **此阶段禁止调用 `Context`**（上下文尚未注入）
- **不建议在此阶段弹出窗口**
- 适合执行轻量的初始化逻辑

```csharp
public override Task Load()
{
    _defaultTimeout = 5000;
    // 禁止：Context.XXX 任何调用
    return Task.CompletedTask;
}
```

---

### OnEnable() — 启用阶段 ⭐

**执行时机**：`Load` 之后，插件被正式启用时调用。

**适用场景**：
- 读取 `Context.Manifest`、`Context.DataPath`
- **订阅所有机器人事件**
- 启动定时任务或后台服务

**注意事项**：
- **所有事件订阅必须在此方法中进行**
- `Context` 已可用，可安全调用
- 同样不建议在此方法弹出窗口（请用 `OnSettings`）

```csharp
public override Task OnEnable()
{
    // 读取插件信息
    _pluginId = Context.Manifest.Id;
    _dataDirectory = Context.DataPath;
    
    // 订阅事件（统一在这里注册）
    Context.Events.On<GroupMessageEvent>(OnGroupMessage);
    Context.Events.On<BotOnlineEvent>(OnBotOnline);
    Context.Events.On<HeartbeatEvent>(OnHeartbeat);
    
    return Task.CompletedTask;
}
```

---

### 事件处理阶段（运行中）

插件启用后，所有订阅的事件会持续触发，直到插件被禁用。

详见 [事件系统](/concepts/events) 章节。

---

### OnDisable() — 禁用阶段

**执行时机**：用户禁用插件或 UniQQ 关闭时调用。

**适用场景**：
- **取消所有事件订阅**
- 释放非托管资源（文件句柄、网络连接）
- 保存运行时数据

```csharp
public override Task OnDisable()
{
    // 取消事件订阅（必须！）
    Context.Events.Off<GroupMessageEvent>();
    Context.Events.Off<BotOnlineEvent>();
    Context.Events.Off<HeartbeatEvent>();
    
    // 保存运行时状态
    SaveState();
    
    return Task.CompletedTask;
}
```

> **重要**：在 `OnDisable` 中取消订阅是**强制要求**。否则可能导致：
> - 插件被重新启用时**重复触发**事件
> - **内存泄漏**

---

### OnSettings() — 设置阶段

**执行时机**：用户在插件管理器中点击「设置」按钮时调用。

**适用场景**：
- 弹出配置窗口
- 修改插件设置

```csharp
public override Task OnSettings()
{
    // 弹出设置窗口
    var configForm = new ConfigForm(_dataDirectory);
    configForm.ShowDialog();
    return Task.CompletedTask;
}
```

---

### Unload() — 卸载阶段

**执行时机**：插件被彻底卸载时调用。

**适用场景**：
- 删除临时文件
- 最终的清理操作

```csharp
public override Task Unload()
{
    // 同同同同样不建议在此函数进行窗口弹出的操作
    // 清理临时文件
    string tempPath = Path.Combine(_dataDirectory, "temp");
    if (Directory.Exists(tempPath))
        Directory.Delete(tempPath, true);
    
    return Task.CompletedTask;
}
```

---

## 完整生命周期代码模板

```csharp
public class MyPlugin : PluginBase
{
    // ========== 元信息 ==========
    public override string Name => "我的插件";
    public override string Version => "1.0.0";
    
    private string _dataDirectory = string.Empty;

    // ========== 生命周期 ==========

    public override Task Load()
    {
        // 轻量初始化（禁止调用 Context）
        return Task.CompletedTask;
    }

    public override Task OnEnable()
    {
        _dataDirectory = Context.DataPath;
        
        // 订阅事件
        Context.Events.On<GroupMessageEvent>(OnGroupMessage);
        
        return Task.CompletedTask;
    }

    private async Task OnGroupMessage(GroupMessageEvent e)
    {
        // 处理消息
    }

    public override Task OnDisable()
    {
        // 取消事件订阅
        Context.Events.Off<GroupMessageEvent>();
        return Task.CompletedTask;
    }

    public override Task OnSettings()
    {
        // 弹出设置窗口
        return Task.CompletedTask;
    }

    public override Task Unload()
    {
        // 清理
        return Task.CompletedTask;
    }
}
```

---
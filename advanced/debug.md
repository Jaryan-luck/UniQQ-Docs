# 调试与测试

插件开发过程中的调试技巧和测试方法。

---

## 日志调试

使用 `Context.WriteLog` 输出带颜色的日志：

```csharp
// 普通信息（白色）
await Context.WriteLog("插件已启动", Color.White);

// 调试信息（灰色）
await Context.WriteLog($"收到消息: {e.Message.RawText}", Color.Gray);

// 警告（黄色）
await Context.WriteLog("配置未找到，使用默认配置", Color.Yellow);

// 错误（红色）
await Context.WriteLog($"API 调用失败: {ex.Message}", Color.Red);

// 成功（绿色）
await Context.WriteLog("操作成功完成", Color.Green);
```


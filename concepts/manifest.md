# 插件发布（plugin.json）

每个 UniQQ 插件必须包含一个 `plugin.json` 用于声明插件的基本信息和加载方式。

---

## plugin.json 完整结构

```json
{
  "id": "MyPluginId",
  "name": "我的插件",
  "author": "作者名",
  "version": "1.0.0",
  "description": "插件描述",
  "runtime": "dotnet",
  "entry": "MyPlugin.dll",
  "sdkVersion": "1.0.0",
  "MinFrameworkVersion": "1.0.0",
  "architecture": "any",
  "website": "http://example.com"
}
```
---

## 字段说明

### 基础信息

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ? | 插件唯一标识符，建议使用英文 + 数字，全局唯一 |
| `name` | string | ? | 插件显示名称，在插件管理器中展示 |
| `author` | string | ? | 作者名称 |
| `version` | string | ? | 插件版本号，建议遵循 `主版本.次版本.修订号` 格式 |
| `description` | string | ? | 插件功能描述 |

### 运行时配置

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `runtime` | string | ? | 运行时类型，目前固定为 `"dotnet"` |
| `entry` | string | ? | 入口 DLL 文件名（编译输出名称），如 `"MyPlugin.dll"` |
| `sdkVersion` | string | ? | 所依赖的 SDK 版本号 |
| `MinFrameworkVersion` | string | ? | 最低框架版本要求 |
| `architecture` | string | ? | 架构兼容性，`"any"` |

### 其他

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `website` | string | ? | 插件官网或项目地址 |

---
## 字段详细说明

### id（插件 ID）

- 用于内部标识，**不可重复**
> **个人观点**
- 建议格式：`作者名.插件名` 或 `命名空间风格`
- 示例：`Lucent.HelloWorld`、`MyPlugin`

### name（显示名称）

- 在 UniQQ 插件管理界面显示的名称

### entry（入口文件）

- 必须是项目编译输出的 DLL 文件名

### sdkVersion 与 MinFrameworkVersion

- `sdkVersion`：此插件开发所用的 SDK 版本
- `MinFrameworkVersion`：插件可运行的最低 UniQQ 框架版本
- 通常两者保持一致

### architecture（架构）

| 值 | 说明 |
|----|------|
| `"any"` | 兼容所有架构（推荐） |
| `"x86"` | 仅 32 位 |
| `"x64"` | 仅 64 位 |
---

## 注意事项
1. **id 变更会影响已安装插件**
   - 如果已安装插件的 `id` 发生变更，UniQQ 会将其视为新插件
   - 建议上线前确认 id 不再变化

2. **entry 必须与编译输出匹配**
   - 如果项目名称为 `MyPlugin`，默认输出为 `MyPlugin.dll`
   - 可在 `.csproj` 中修改 `<AssemblyName>` 来改变输出名称

3. **编码格式**
   - 保存为 **UTF-8 无 BOM** 格式
   - 避免使用特殊不可见字符

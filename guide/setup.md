# 环境配置

详细的开发环境搭建指南。

---

## 1. 安装 .NET SDK

UniQQ 插件基于 .NET 10.0 开发，需安装对应 SDK。

### 下载

- 官网下载：[dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet/10.0)
- 安装后验证：

```bash
dotnet --version
dotnet --list-sdks
```

### 关于目标框架

插件项目必须使用 `net10.0` 框架：

```xml
<TargetFramework>net10.0</TargetFramework>
```

---

## 2. 安装 IDE（任选其一）

### Visual Studio IDE

- 安装时勾选 **.NET 桌面开发** 工作负载
- 确保包含 **.NET 10.0 SDK**

### JetBrains Rider（推荐）

- 直接打开 `.csproj` 文件即可
- Rider 会自动检测 SDK 并还原依赖

---

## 3. 配置 SDK 引用

### 获取 UniQQ.SDK.dll

### 在项目中引用

```xml
<ItemGroup>
  <Reference Include="UniQQ.SDK">
    <HintPath>(UniQQSDK的路径)</HintPath>
  </Reference>
</ItemGroup>
```

> 建议将 SDK DLL 放在项目目录外统一管理，多个插件可共用同一引用。

---

## 4. 编译测试

```bash
# 还原依赖
dotnet restore

# 编译（Debug 模式）
dotnet build

# 编译（Release 模式）
dotnet build -c Release
```

编译成功后会生成 `你的插件名.dll`，配合 `plugin.json` 打包，即可安装到 UniQQ。

---

## 7. 常见环境问题

| 问题 | 解决方法 |
|------|----------|
| `NETSDK1045` 当前 .NET SDK 不支持目标框架 | 安装 .NET 10.0 SDK |
| `CS0246` 找不到 UniQQ.SDK | 检查 DLL 引用路径是否正确 |
| `CS0012` 类型定义缺失 | 确保 SDK 版本与插件 `sdkVersion` 匹配 |

---

## 下一步

环境搭好后，建议从 [快速上手](/guide/quickstart) 开始编写你的第一个插件。

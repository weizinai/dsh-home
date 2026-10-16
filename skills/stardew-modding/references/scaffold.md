# 从零新建一个 SMAPI 模组

供 `stardew-modding` 技能加载。基准与冲突处理同 `SKILL.md`。

本文只管从零建项目文件。建好之后往里放什么——各职责目录的准入判据与 `ModEntry` 接线次序——见仓库的 [模组目录与职责约定](../../../../docs/directory-layout.md)。

## 一个最小可运行模组 = 三个文件

都在模组目录根下：

```
<Name>.csproj
manifest.json
ModEntry.cs
```

## csproj：空库项目 + 一个 NuGet 包

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>net6.0</TargetFramework>
        <Version>1.0.0</Version>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Pathoschild.Stardew.ModBuildConfig" Version="4.4.0"/>
    </ItemGroup>
</Project>
```

- 官方只要求两步：建一个**空库项目**、引用 `Pathoschild.Stardew.ModBuildConfig`。
- 这一个包负责整个构建期（构建即部署、发布 zip、把 manifest 里的 `%ProjectVersion%` 换成 csproj 的 `<Version>`）：属性与开关见 `references/mod-package.md`。
- 改完就 `dotnet build <模组目录>`：编译完自动部署进游戏 `Mods`，直接进游戏验证，不用手动拷贝。

- **别照抄 SMAPI 源码树里 `src\SMAPI.Mods.*` 的 csproj**：它们是 SMAPI 仓库的内部构建（`ProjectReference` 直连 `SMAPI.csproj` + `Import ..\..\build\common.targets` + 手写游戏程序集 `Reference`），既不是 ModBuildConfig 的用法，拿去用还会因为那个 `Import` 路径不存在直接构建失败。

## manifest.json：三个必填 + 二选一

```json
{
    "Name": "模组名",
    "Author": "作者名",
    "Version": "%ProjectVersion%",
    "Description": "一句话说明这个模组做什么。",
    "UniqueID": "作者名.模组名",
    "EntryDll": "模组名.dll"
}
```

- **运行时真正强制的只有三个**：`Name`、`Version`、`UniqueID`。`Version` 写 `0.0.0` 也会加载失败，但拦它的是语义版本解析（全零版本非法），不是 manifest 校验器。
- **`EntryDll` 与 `ContentPackFor` 二选一，必须给且只能给一个**：C# 模组填 `EntryDll`，内容包填 `ContentPackFor`。同时给、或都不给，都会加载失败。
- **官方 JSON schema 另外要求 `Author` 与 `Description`**，运行时的校验器却不查这两个——所以"模组能加载"和"过 schema 校验"是两个标准。两个都填。
- **`UniqueID` 只能用字母、数字、下划线、点、连字符**，惯例写成 `作者.模组名`。
- **字段名统一写 `UniqueID`（大写 ID）**。运行时大小写不敏感，写 `UniqueId` 也能正常加载；但官方 schema 的属性名是精确匹配且禁止多余字段，用 schema 校验时 `UniqueId` 会被判成多余字段。
- **`MinimumApiVersion` 可选，但填高了模组会直接加载失败**（不是警告），提示 "it needs SMAPI x.y.z or later"。
- `EntryDll` 必须是 `xxx.dll` 形式，且那个 DLL 真的在模组目录里；`UpdateKeys` 用 `Nexus:20685`、`GitHub:用户/仓库` 这类格式，可加 `@子键`。

## 入口类

```csharp
using StardewModdingAPI;

namespace 你的命名空间;

internal class ModEntry : Mod
{
    public override void Entry(IModHelper helper)
    {
        // 在这里做初始化：读配置、注册事件、打补丁
    }
}
```

- `Entry(IModHelper)` 是**唯一必须的 override**，`EntryDll` 指向的程序集里要有这个类。
- **入口类的类名、命名空间随意，`internal` 也可以**：SMAPI 是扫 DLL 里所有 `Mod` 子类来认入口的，所以**恰好一个**才行，两个就报 `its DLL contains multiple 'Mod' subclasses.`。
- `Mod` 基类已经给了 `Helper`、`Monitor`、`ModManifest` 三个属性，别再自己造一个 helper。

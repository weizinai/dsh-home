# 构建、部署与打包

供 `stardew-modding` 技能加载。基准与冲突处理同 `SKILL.md`。

## 一个包管住整个构建期

引 `Pathoschild.Stardew.ModBuildConfig` 之后，`.csproj` 里不用手写游戏程序集与 SMAPI 的 `Reference`，构建期的事全归它：

- **构建即部署**：每次构建把模组拷进 `$(GameModsPath)`（默认 `$(GamePath)\Mods`），构建完直接进游戏验证。
- **发布 zip**：每次构建同时生成 `<ModFolderName> <版本>.zip`——`ModFolderName` 默认取 **MSBuild 项目名**（不是 manifest 的 `Name`），非法文件名字符替换成 `.`；默认落在构建输出目录 `$(TargetDir)`，即 `bin\<配置>\<目标框架>\`，**不是**字面 `bin`。
- **版本号替换**：manifest 里的 `%ProjectVersion%` 构建时替换成 csproj 的 `<Version>`（整份文件、大小写不敏感），部署副本与 zip 内都已替换——所以版本号只改 csproj。
- **调试**：构建好的项目可在 Visual Studio 里直接启动、调试游戏（仅 Windows；Linux/macOS 因 Mono 包装限制不可）。

游戏路径探测失败或本机有多个安装时，在 `%USERPROFILE%\stardewvalley.targets`（非 Windows 是 `$(HOME)\stardewvalley.targets`）里写：

```xml
<Project>
    <PropertyGroup>
        <GamePath>游戏目录</GamePath>
    </PropertyGroup>
</Project>
```

## 属性放哪

三处，从宽到窄：用户级 `stardewvalley.targets`（本机所有模组项目）→ 仓库级 `Directory.Build.props`（该目录树下所有项目，多模组仓库的共享默认值放这里）→ 项目 `.csproj`（单个模组的特例）。

包的默认值都带 `Condition="'$(X)' == ''"`，上面任何一处设了都能盖掉默认值。

**属性不能先用后定义**：`GameModsPath` 依赖 `GamePath`，同一个 `PropertyGroup` 里它必须写在后面。个别属性（如 `BundleExtraAssemblies`）在包 targets 导入的那一刻就被读取，所以只能定义在项目 `.csproj` 或 `Directory.Build.props` 里——写进 `Directory.Build.targets` 等于没设。

## 开关

默认值都好用，要改才写。

| 属性 | 默认 | 什么时候改 |
|---|---|---|
| `GamePath` / `GameModsPath` | 探测结果 / `$(GamePath)\Mods` | 探测失败、有多个安装，或要把模组装进 `Mods` 下的子目录 |
| `EnableHarmony` | `false` | **用 Harmony 打补丁就必须置 `true`**——`0Harmony` 引用由它加（别自己引 `Lib.Harmony` 或手写 `0Harmony` `Reference`，会和 SMAPI 自带的那份打架） |
| `EnableModDeploy` / `EnableModZip` | `true` | 不想每次构建都部署 / 都出 zip 时置 `false` |
| `ModFolderName` | 项目名 | 要让游戏里与 zip 里的文件夹名不同于项目名 |
| `ModZipPath` | `$(TargetDir)` | 要把 zip 收进别处（写**目录**——填成 `…\x.zip` 会先建出一个叫 `x.zip` 的目录再往里放） |
| `IgnoreModFilePaths` / `IgnoreModFilePatterns` | 空 | 有文件不该进部署与 zip：前者是逗号分隔的字面路径（**大小写敏感**），后者是正则 |
| `BundleExtraAssemblies` | 空 | 要连第三方 DLL 一起打包时写 `ThirdParty` |

`BundleExtraAssemblies` 只认 `ThirdParty`、`System`、`Game`（可逗号组合，如 `Game,System,ThirdParty`；**大小写敏感**）。**`All` 认不出来**：任务只发一条 warning 说该值被忽略，游戏程序集与 SMAPI 因此进不了部署与 zip，实际效果约等于 `System,ThirdParty`——上游文档"`All` 等价于 `System, Game, ThirdParty`"这句是错的。要把游戏程序集也带上，必须显式写 `Game,System,ThirdParty`，并且开了这项就得人工复查哪些文件真的被部署出去。

## 捆绑内容包

主模组可以带任意多个内容包一起发布：构建时自动套一层以 `ModFolderName` 命名的父目录，主模组与各内容包各占其下一层（zip 内、部署处都是这个结构）。

```xml
<ItemGroup>
    <ContentPacks Include="content packs/[CP] SomeContentPack" Version="$(Version)"/>
</ItemGroup>
```

- `Include`、`Version` 必填；`Version` 通常写 `$(Version)` 与主模组保持一致——包拿它比对内容包 manifest 的版本，不符直接构建失败。
- 可选元数据：`FolderName`（该包在父目录下的文件夹名，默认取 `Include` 的目录名）、`ValidateManifest`（默认 `true`；**置 `false` 只跳过版本比对**，manifest 缺失或非法照样失败）、`IgnoreModFilePaths` / `IgnoreModFilePatterns`（语义同开关表）。

## 构建时的代码警告

包里带 SDV 专用分析器，两条规则默认都是 Warning 级：`AvoidNetField`（访问了 net 字段而游戏有等价属性）、`AvoidObsoleteField`（引用了过时字段）。隐藏警告用这两个 ID，不是 `SMAPI00xx` 形式。

另有一条 `Wrong processor architecture`——它是 MSBuild 警告，不是分析器规则。

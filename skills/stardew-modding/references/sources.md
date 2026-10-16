# 本机参考资料来源

写、审本仓库模组代码时的查证去处。本文件只回答「去哪查」与「各来源管什么」；查证顺序（本地优先，再官方）见仓库 `AGENTS.md`。下面路径都在本机。

## 1. 游戏本体反编译源码 — `D:\Projects\dotnet\StardewValleyModsReference\StardewValley\StardewValley`

解剖游戏内部行为的最终依据：字段、方法、事件、菜单、NPC、地点、Netcode 都在这里。锚点是 `StardewValley/Game1.cs`（主类），功能按命名空间分布（`.Menus` `.Objects` `.Locations` `.Characters` `.Events` `.Netcode` `.Buffs` `.Enchantments`）。

- **版本号没有源码常量**：`Game1.version` 等字段由程序集 `AssemblyInformationalVersionAttribute` 在 `Game1` 构造函数里运行时填充，反编译源码里读不到字面值。要具体版本事实就查 wiki 迁移指南表或运行时值。

## 2. 游戏本体数据资产（解包）— `D:\Projects\dotnet\StardewValleyModsReference\Content`

游戏 `Content` 目录的解包产物：`Data/` 顶层是数据表 JSON（`Locations.json`、`Objects.json`、`Characters.json`、`Machines.json`、`Shops.json` 等），其下另有 `Events/`、`Festivals/`、`TV/`；`Maps/` 是 .tmx 与 .png；`Strings/` 是本地化字符串；其余目录按资产类型划分（`Characters/`、`Portraits/`、`TerrainFeatures/`、`TileSheets/` 等）。

用途是查**数据资产的字段与取值**——某处地点的 `AlwaysActive`、某件物品的价格、某台机器的产出规则，游戏运行时加载的就是这些文件。与源码的分工：源码答「这段逻辑怎么写」，这里答「这份数据里有什么」；`Data/<资产名>` 的加载器在源码里（如 `DataLoader.Locations`），取值在这里。

## 3. SMAPI 源码与文档 — `D:\Projects\dotnet\StardewValleyModsReference\SMAPI`

SMAPI 规则、接口、事件的权威来源，兼官方总结文档。

- 锚点：`src/SMAPI/IModHelper.cs`（`IModHelper` 接口）、`src/SMAPI/Translation.cs`（**类名是 `Translation`，不是 `ModTranslation`**——`ModTranslation` 只是 NuGet 包名）、`src/SMAPI/Framework/ModHelpers/`（各 helper 实现）。
- 文档：`docs/README.md`（总索引，含 "For modders" 一节）。
- **构建与打包配置**：`docs/technical/mod-package.md`（`docs/mod-build-config.md` 全文只是一行重定向，别指它）；与所钉版本严格匹配的副本在 NuGet 缓存 `%USERPROFILE%\.nuget\packages\pathoschild.stardew.modbuildconfig\<版本>\mod-package.md`。文档与包实际行为冲突时，以包里 `build\*.targets` 与 DLL 的行为为准。

## 4. 官方 wiki 模组索引 — https://stardewvalleywiki.com/Modding:Index

定位「某主题有没有官方文档、常见做法是什么」。结构是 Using mods / Creating mods（**C# 模组**与**内容包**两条路线）/ Specific topics（游戏机制、NPC、物品数据、地点与地图、迁移指南等）/ See also（兼容性列表、log parser、JSON 校验等工具）；细节进各专题页。

## 5. Harmony 文档 — https://harmony.pardeike.net/（Harmony 2）

写、审 Harmony patch（Prefix/Postfix/Transpiler/Finalizer）时查这里。

- 关键限制：只能 patch 有 IL 方法体的方法（含构造器与属性访问器）；过小的方法会被**内联**导致 patch 不生效；不能给类加字段、不能扩展枚举；泛型方法或泛型类内的 patch 可能不按预期工作。
- 两种打法：**注解式**（`[HarmonyPatch]` + 静态 `Prefix`/`Postfix` 方法）或**手动反射式**（`harmony.Patch(...)` + `AccessTools.Method`/`SymbolExtensions.GetMethodInfo`）。
- API 结构：Basics / Targeting / Attributes / Transpiling / Helper classes 分组，`HarmonyLib` 命名空间的完整类表（`AccessTools`、`CodeInstruction`、`CodeMatcher`、`Traverse`、`SymbolExtensions` 等）。

## 场景查表

| 场景 | 去查 |
| --- | --- |
| 定位游戏内类 / 事件触发点 / 数据入口 | 反编译源码（第 1 节，按命名空间） |
| 查原版数据资产的字段与取值 | 解包数据（第 2 节，`Data/*.json`） |
| 调 SMAPI（事件、helper、资产） | 本地 SMAPI 源码（第 3 节） |
| 构建、发布、捆绑内容包 | `mod-package.md`（第 3 节，认 NuGet 缓存里与所钉版本一致的那份） |
| 写内容包 / 数据格式 | wiki 专题页（在线，从索引页进入） |
| 写 / 审 Harmony patch | Harmony intro + API（在线，第 5 节） |
| 确认版本 / 迁移事实 | wiki 迁移指南表（在线）或运行时 `Game1.version` |

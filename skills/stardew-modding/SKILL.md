---
name: stardew-modding
description: 星露谷（Stardew Valley）SMAPI 模组开发的通用知识库：新建模组的骨架与 manifest、Harmony 补丁、SMAPI 事件与 API、调试排错、内容包与 Content Patcher。写或改 SDV/SMAPI 模组时加载。
---

# Stardew Valley 模组开发（通用知识）

**基准**：本技能的知识以本机源码树为准——本机参考源码都在 `D:\Projects\dotnet\StardewValleyModsReference\` 下，各来源的具体路径见 `references/sources.md`。发现哪一条与实际源码冲突时，**以源码为准，就地改写那一条**（就地修订），并在当次回复里报告**改了哪一条、依据是什么**——改完不报告，等于替别人做了他没批准的修改。

## 遇到这类问题，读哪个文件

| 问题 | 读 |
|---|---|
| 从零新建一个 SMAPI 模组：csproj、manifest.json、入口类 | `references/scaffold.md` |
| 写/改模组的文案与翻译键：`i18n\` 文件格式、locale 回退、代码里怎么取 | `references/translation.md` |
| 构建、部署与打包：ModBuildConfig 的属性与开关、发布 zip、捆绑内容包 | `references/mod-package.md` |
| 查证某个事实该去哪找：反编译源码、解包数据、SMAPI 源码与文档、wiki、Harmony 各自的路径 | `references/sources.md` |

以后每验收一个新的知识领域，就在上表加一行。表里只放"该不该读、读哪个"的判据，不放内容。

## 先查证，再动笔

写进代码或注释时，注明来源。各来源的路径、各管什么、以及「这个场景去查哪」的对照表都在 `references/sources.md`，不要凭记忆。

## 边界

- 本技能只收**与具体框架无关**的通用知识。判据：这条事实**换一个模组、换一个作者仍然成立**。
- 只有「本仓库某模组要接管或拦截某个行为」才需要的事实不算通用——那属于该模组的机制，归它的 `docs/architecture.md`，不进这里。
- 若当前仓库有 PiCore 这类自建框架，**优先复用它的基建**；它的用法以该仓库自己的文档为准，不写进本技能。

## 一条知识值不值得收

**下次遇到同类问题时，它能让我少走一次弯路**——通不过就不收，不确定默认不收。写之前把事实在源码或在线文档里核实过；出处是写的依据，不必抄进文件。

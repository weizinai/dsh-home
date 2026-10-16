# 文案与翻译键

供 `stardew-modding` 技能加载。基准与冲突处理同 `SKILL.md`。

## i18n\ 目录：模组可以没有它

- 不碰翻译的模组不需要 `i18n\`；要文案时在模组目录下建它，**每个 locale 一个文件**，`default.json` 是英文基文案兼兜底。
- 文件是**扁平 key ⇒ string**；locale 缺失时逐级回退（`pt-BR.json` < `pt.json` < `default.json`）。
- 另一种布局 `i18n\<locale>\<file>.json`（子目录）；**两种布局不能混用**，混用时只取顶层文件并告警。
- 值**只能是字符串**，但**允许 `//` 注释和尾逗号**。

## 代码里取键：用生成的类，别手写字符串

- 引 NuGet 包 `Pathoschild.Stardew.ModTranslationClassBuilder`，由 `i18n/default.json` 生成静态类（默认名 `I18n`）；`Entry` 里加一行 `I18n.Init(helper.Translation);`，之后代码任意位置直接用 `I18n.*`，改了 json 自动跟着更新。
- **引了这个包，`i18n/default.json` 就成了硬要求**：文件缺失或为空，构建直接失败（Error 级，不是警告）。不碰翻译的模组就别引它。
- 键 → 成员名：转 CamelCase、`.` 变 `_`（`ready` → `I18n.Ready()`、`ready-now` → `I18n.ReadyNow()`、`generic.ready-now` → `I18n.Generic_ReadyNow()`）；编译期校验的是**键名、方法名与 token 名**，token 的**类型不校验**。
- 写错名字当场编译报错，而不是进游戏跑到那一行才炸。
- 它是**独立第三方包，不是 SMAPI 的 API**——在 SMAPI 源码里搜它是搜不到的。

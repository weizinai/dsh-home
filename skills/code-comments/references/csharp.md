# C# 注释

本文件补 [../SKILL.md](../SKILL.md) 第 6 节的语言特有规则，只写 C# 的记法与补充规则；成员注释义务见核心第 4 节，本文件不重述。

## 文档注释

- **必须**：用中文 XML，`///` 而非 `//`。
- **必须**：`<summary>` 写契约（做什么、边界）；签名已表达的部分不重复。
- **必须**：读文档才知道的额外约定与已知局限写进 `<remarks><para>`。
- **必须**：交叉引用用 `<see cref="…"/>`，不重复抄名字。
- **必须**：XML 里写代码片段用 `<c>` 包住，`<` `>` `&` 转义成 `&lt;` `&gt;` `&amp;`。
- **必须**：私有方法里复杂逻辑用实现注释说明意图。
- **必须**：`#region 名字` 与 `#endregion` 成对；可用范围见核心第 5 节。

抑制警告与空 `catch` 的理由注释义务见 `cs-code-style` 技能，级别以该技能为准，本文件不重述。

缺 `<summary>` 报 CS1591；仓库开了 `GenerateDocumentationFile` + `TreatWarningsAsErrors` 时它是编译错误。

## Unity 差异

- **必须**：真正对外的运行时 public / protected API 写中文 XML，`<summary>` 必写；签名已完全表达语义的成员（豁免范围见核心第 4 节）除外。`<param>` / `<returns>` / `<exception>` 按需。仅给 Inspector 看的序列化字段不强制 XML。
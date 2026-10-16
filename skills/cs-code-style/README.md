# cs-code-style

C# 风格规范。`SKILL.md` 只写 `.editorconfig` 强制不了的规则；机器强制的格式与命名在 `reference/.editorconfig`。

## 文件

| 文件 | 用途 |
| ---- | ---- |
| `SKILL.md` | 工具管不到的必须 / 建议；写或审 C# 时加载 |
| `reference/.editorconfig` | 可复制的编辑器/编译器模板 |
| `README.md` | 启用说明 |

## 用到 C# 项目里

把模板复制到项目根，.NET SDK / VS / Rider 会读取：

```bash
cp reference/.editorconfig /path/to/your/project/.editorconfig
```

生成时把 IDE 规则当警告（否则 `dotnet build` 默认不报）：

```xml
<EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
```

生成代码加一段 `generated_code = true`，不要改模板本身。

校验：

```bash
dotnet build
```

必须零警告。

## 改规则

能用 `.editorconfig` 表达的改 `reference/.editorconfig`，再同步各项目根副本；不能表达的改 `SKILL.md`。不要在技能正文复述已进 `.editorconfig` 的项。

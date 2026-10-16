---
name: cs-code-style
description: C# 代码风格。写或审 C#、改 .editorconfig 时加载。
---

# cs-code-style：C# 代码风格

写或审 C# 时「必须」级一律遵守；「建议」级在可行时遵守。规则未覆盖的：同文件已有代码优先；新文件用本规范。

机器强制的格式与命名以项目根 `.editorconfig` 为准（必须级 = warning）。从 `reference/.editorconfig` 复制。生成时强制需在 csproj / `Directory.Build.props` 设 `<EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>`。生成代码标 `generated_code = true`。

第一遍就按编译器会拒的方式写：实例成员加 `this.`；方法用块体；局部变量一律 `var`；文件范围命名空间；预定义类型用关键字（`string` / `int`）。

## 1. 工具链

- **必须**：`dotnet build` 零警告。警告当缺陷修。确需 `#pragma warning disable` 或项目级 `NoWarn` 时，紧邻注释说明理由。
- **建议**：新项目 `<Nullable>enable</Nullable>`。

## 2. 文件

- **必须**：每个文件一个核心类型，文件名与主类型同名且 PascalCase。紧密相关的私有嵌套类型可同文件。
- **建议**：命名空间与项目/目录结构一致。

## 3. 命名

大小写、`I` / `T` / `Async` 前后缀由 `.editorconfig` 强制。此处只管它管不到的：

- **必须**：两字母缩写全大写（`IOStream`），三字母及以上仅首字母大写（`XmlWriter`）；camelCase 同样（`ioStream`、`xmlWriter`）。
- **必须**：名字写全，且不重复类型已表达的信息（`List<TodoItem> items`）。
- **必须**：局部常量 PascalCase（`const int MaxRetries = 3;`）。
- **必须**：特性类型以 `Attribute` 结尾（`RetryAttribute`）。
- **必须**：`class` / `struct` 主构造函数参数 camelCase；`record` 位置参数 PascalCase（会成为公共属性）。
- **必须**：标识符不含连续 `__`（编译器保留）。
- **建议**：普通枚举用单数名词，`[Flags]` 用复数。
- **建议**：泛型类型参数用描述性名字，约束能写进名字就写（`TSession`）；单字母仅当自我说明（`List<T>`）。
- **建议**：除简单循环计数器外不用单字母名。
- **建议**：布尔属性/方法用 `Is` / `Has` / `Can` 前缀。

## 4. 类型与成员

写一个成员时这些同时成立：

- **必须**：能 `const` 就 `const`；不能（`Guid`、`DateTime`、运行时值）用 `static readonly`。非自解释的字面量提取为具名常量或枚举（`private const int SecondsPerDay = 86400;`）。`0`、`1`、`-1` 保持字面量。
- **必须**：值语义才用 struct（`Vector3`、`TimeSpan` 类）；其余用 class。
- **必须**：优先标准异常（`ArgumentException`、`ArgumentNullException`、`InvalidOperationException`、`NotSupportedException`）。异常表达失败，不表达常规分支。`catch` 接到能处理的具体类型；空 `catch` 或抓 `Exception` 时紧邻注释说明为何可忽略。
- **必须**：`async void` 仅限事件处理器；同步与异步成对时同步方法在前。
- **必须**：`out` 只用于纯返回多个值，且放在参数列表最后。`ref` 只用于调用后调用方必须看到的就地修改。
- **必须**：LINQ 用成员扩展方法（`items.Where(...)`）；长链拆中间变量；超过几条语句或多处复用的 lambda 提为具名方法。
- **建议**：集合入参用最宽可用类型（`IEnumerable<T>` / `IReadOnlyCollection<T>` / `IReadOnlyList<T>`）；出参转移所有权用 `IList<T>`，否则返回可承诺的最受限类型（`IReadOnlyList<T>`）。
- **建议**：成员排序为嵌套类型 / 枚举 / 委托 → 常量 → 字段 → 属性 → 构造器 → 方法；组内 public → internal → protected internal → protected → private。
- **建议**：不可变数据用 `record` / `init`。
- **建议**：参数一般不超过 3 个；再多就做成选项对象。
- **建议**：单行只读属性可用表达式体（`public int Id => this.id;`）。
- **建议**：公共入口参数用 `ArgumentNullException.ThrowIfNull(param)`。
- **建议**：方法体约 30 行内；超出时拆。成员之间一个空行；一组重载可紧邻。

## 5. 语句

- **必须**：字符串用插值（`$"Hello, {name}!"`）；少量短片段可用 `+`；循环里反复拼接用 `StringBuilder`。
- **必须**：非穷尽 `switch` 写 `default`（可为空）。枚举穷尽式 `switch` 交给编译器。
- **必须**：每行一条语句；每条语句至多一次赋值。
- **建议**：新委托用 `Func<>` / `Action<>`，不新声明 `delegate`。
- **建议**：布尔条件用 `&&` / `||`，不用 `&` / `|`。
- **建议**：静态成员用声明它的类型名限定（`Foo.Bar`），不用实例或派生类名。
- **建议**：一个声明语句只声明一个变量。
- **建议**：字符串比较写明 `StringComparison`（`string.Equals(a, b, StringComparison.Ordinal)`）。

## 写完自查

对改过的每个 C# 文件：`dotnet build` 零警告，且第 2–5 节每条「必须」成立。`.editorconfig` 已覆盖的项以编译输出为准。

## 改 .editorconfig 时

共享规则改 `reference/.editorconfig`，再同步到项目根（项目特有的 `generated_code` 排除留在项目根）。必须级 → `warning`，建议级 → `suggestion`。能用属性表达的只改 `.editorconfig`，不要写进本节以上的正文。

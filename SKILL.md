---
name: typst-cn
description: 用 Typst 构建中文学术论文/汇报文稿。触发条件：用户要求用 Typst 排版中文文档、创建 typst 文件、编译 typst、写中文论文或汇报的 typst 稿子。
version: 1.2.0
---

# Typst 中文文档构建

基于实战踩坑总结的 Typst 中文排版最佳实践。

## 字体设置

中英文混排使用字体回退数组：英文在前作为主字体，中文在后作为回退：

```typst
#set text(font: ("Times New Roman", "Songti SC"), size: 12pt, lang: "zh")
```

Mac 上中文宋体为 `Songti SC`，黑体为 `Heiti SC`。数学字体 Typst 自动选用 `NewCMMath-Book`，无需手动设置。

## 段落设置（最核心）

```typst
#set par(
  leading: 0.8em,
  first-line-indent: (amount: 2em, all: true),
  spacing: 0.8em,
)
```

各参数说明：
- `leading`：行距（基线到基线），0.8em 适合 12pt 正文
- `first-line-indent`：**必须用 `(amount: 2em, all: true)` 格式**。`all: true` 是关键——让标题后首段、公式后段落、图表后段落也缩进，无需任何 `#box[]` hack
- `spacing`：段落间距，建议 ≥ leading，否则段间距看起来比行间距还小

### 两端对齐（按需）

学术论文或正式报告通常需要两端对齐。**在用户明确需要时使用**：

```typst
#set par(justify: true)
```

注意：启用 `justify: true` 后 Typst 会在单词间拉伸空格，中文文本由于词间无空格，效果通常已经不错。中英混排时若英文单词间距过大，可配合 `hyphenate: true`（需要 `#set text(hyphenate: true)`）。

## 标题层级

**封面大标题不要用 `=`**——`=` 会被 Typst 自动编号为 "1"，导致后续 `=` 标题变成 "2"、"3"，`==` 变成 "2.1"、"2.2"。封面用显式格式化：

```typst
#align(center)[
  #text(size: 18pt, weight: "bold")[论文标题]
  #v(0.3em)
  #text(size: 13pt, style: "italic")[English Title]
]
```

正文标题用 `=` 和 `==`：

```typst
#set heading(numbering: "1.")
= 一级标题   // 编号：1, 2, 3...
== 二级标题  // 编号：1.1, 1.2, 2.1...
```

**标题文本中不要手写编号**（如"一、"、"2.1"），Typst 自动生成。

## 加粗

Typst 用 `*文字*` 加粗（**不是** Markdown 的 `**文字**`）：

```typst
*这是粗体*    // 正确
**这是粗体**  // 错误：被解析为嵌套的加粗星号，可能不报错但显示异常
```

这是 Typst 与 Markdown 之间的一个常见混淆点。

## 目录

```typst
#outline(title: "目录", depth: 3)
```

`depth` 控制显示到第几级标题。通常放在封面后、正文前，配合 `#pagebreak()` 使用。`title` 参数可自定义目录标题文字。

## 分页

```typst
#pagebreak()
```

常用场景：封面后、目录后、参考文献前。Typst 会自动处理正文分页，`#pagebreak()` 仅用于显式断页。

## 表格

表格用 `#figure(table(...), caption: ...)` 包裹，与图片一致，Typst 自动编号：

```typst
#figure(
  table(
    columns: (auto, auto, auto),
    align: (left, left, right),
    [*表头1*], [*表头2*], [*表头3*],
    [数据1], [数据2], [数据3],
    [数据4], [数据5], [数据6],
  ),
  caption: "表格描述"
) <tab:label>
```

要点：
- `columns`：各列宽度，`auto` 为自适应。也可用具体值如 `columns: (3cm, 1fr, 2fr)`
- `align`：各列对齐方式（`left` / `center` / `right`）
- 表头加粗用 `[*...*]`（在 `[]` 内容块中加粗）
- `caption` 中用 `*...*` 写强调文字

## 图表

图片同样用 `#figure(...)`，caption 中不要手写"图 1"、"表 1"，Typst 自动编号：

```typst
#figure(
  image("figure.png", width: 90%),
  caption: [*简短描述。* 详细说明...],
) <fig:label>
```

### 交叉引用

用 `<label>` 定义标签，用 `@label` 引用。Typst 自动替换为对应编号：

```typst
// 定义
#figure(...) <fig:results>
#figure(...) <tab:summary>

// 引用
@fig:results 展示了实验结果。
如 @tab:summary 所示，各改进步骤均有正增益。
```

命名规范：图表标签用 `<fig:xxx>` 和 `<tab:xxx>` 前缀区分，便于管理。引用时 Typst 会自动显示"图 1"或"表 2"。

## 数学公式

### 多字母标识符加引号

Typst 数学模式中，多字母标识符会被当作单字母乘积。必须加引号：

```typst
$"RV"_t$     // 正确：变量 RV，下标 t
$RV_t$       // 错误：被解析为 R × V_t
```

### 显示公式

直接用 `$ ... $`（`$` 两边有空格即为显示数学）。配搭 `all: true` 后，公式后段落缩进不受影响，无需额外包装：

```typst
$ "RV"_t = sum_(i=1)^n r^2 $
```

### 数学模式注意事项

- `%` 是 Typst 数学模式中的注释符，不能直接用于百分号。需要写作 `\%`
- 乘号用 `times` *而非* `\times`（Typst 数学命令不带反斜杠，这是 LaTeX → Typst 最常见的迁移坑）
- 除法用 `\/` 而非 `/`（`/` 在数学模式中被视为连字符）
- **LaTeX 迁移注意**：Typst 数学命令均不加 `\` 前缀。常见对照：`\times`→`times`、`\frac{a}{b}`→`a/b`、`\sqrt`→`sqrt`、`\sum`→`sum`

### 数学模式上标/下标分组

Typst 用 `(...)` 而非 `{...}` 进行分组：

```typst
$ (1+"WACC")^(t-0.5) $   // 正确：括号分组
$ (1+"WACC")^{t-0.5} $   // 错误：花括号在 Typst 数学中非分组符
```

## 文本中避免的坑

### `@` 符号需转义

`@` 在 Typst 中是引用语法（`@label`），文本或数学模式中的 `@` 会被解析为引用。像 `MAP@12` 这样的指标名必须转义：

```typst
MAP`@`12    // 正确：用反引号包裹 @
MAP@12      // 错误：@12 被解析为标签引用，编译报错
```

数学模式中同理：

```typst
$ "AP"`@`12 $    // 正确
$ "AP"@12 $      // 错误
```

### 其他坑

- **美元符号**：文本中的 `$` 必须转义 `\$`，否则 Typst 解析为数学模式
- **段间距**：不要在每个段落间加 `#v()`——代码冗余且间距不可控。统一用 `#set par(spacing: ...)`。`#v()` 仅用于章节间大分隔
- **`#box[]` hack**：不要用。`all: true` 一行解决所有缩进问题

## 页面设置（按需）

**在用户明确需要自定义页面尺寸或边距时使用**。默认页面设置通常已满足需求。

```typst
#set page(margin: (x: 2.5cm, y: 2.5cm), numbering: "1")
```

- `margin`：`(x: ..., y: ...)` 分别控制左右和上下边距；也可用 `(top: ..., bottom: ..., left: ..., right: ...)` 分别指定
- `numbering`：页码格式，`"1"` 即阿拉伯数字；设为 `none` 则不显示页码

## 文档元信息（按需）

**用户需要设置文档标题/作者时使用**。对于一些正式的报告或论文，可以设置以下元信息：

```typst
#set document(title: "论文标题", author: "作者名")
```

这些信息会嵌入 PDF 元数据中，通常不影响正文排版。如果需要封面上显示标题和作者，需另外在正文中排版（见标题层级一节）。

## 列表缩进（按需）

**当默认列表缩进距离不合适时使用**：

```typst
#set enum(indent: 1em)   // 有序列表缩进
#set list(indent: 1em)   // 无序列表缩进
```

Typst 使用 `+` 开头表示无序列表（bullet），`-` 开头也表示列表项。有序列表使用 `+` 或数字。缩进值可按需调整。

## 页码控制

**封面不计入页码，正文从第 1 页开始**：

```typst
// 封面部分
#set page(numbering: none)
#align(center)[
  ...封面内容...
]
#pagebreak()

// 正文开始——重置页码计数器
#counter(page).update(1)
#set page(numbering: "1")
```

关键点：`#counter(page).update(1)` 将当前页的计数器设为 1，后续 `#set page(numbering: "1")` 让页码以阿拉伯数字显示。

## 内容块闭合

`#align(center)[...]`、`#figure(...)[...]` 等块级元素的内容块 `[...]` 必须闭合。常见错误：

```typst
#align(center)[
  #text(size: 20pt)[标题]
  #v(2cm)
  #text(size: 10pt)[副标题]
// ❌ 缺少 ]
#pagebreak()
```

正确写法：
```typst
#align(center)[
  #text(size: 20pt)[标题]
  #v(2cm)
  #text(size: 10pt)[副标题]
]  // ✅
#pagebreak()
```

Typst 的错误信息会指向 `#align(center)[` 处报告"unclosed delimiter"。

## 表格注意事项

### 不支持单元格合并

Typst 原生的 `table()` 不支持 `colspan`/`rowspan`。如需复杂布局，拆分为多个简单表（每个表每行单元格数相同）：

```typst
// ❌ 不要：试图在同一表中左右并排两组不同列的数据
// ✅ 改为：拆成两个独立的 #figure(table(...), ...) 分别展示
```

### 空内容块占位

表格中需要留空时使用空内容块 `[]`，不要省略该位置。每行的单元格数必须与 `columns` 数量严格一致。

## BibTeX 文献管理

Typst 通过 `#bibliography("file.bib")` 支持 BibTeX/BibLaTeX 格式。`.bib` 文件注意事项：

- 注释用 `%`，**不是** `//`（与 Typst 代码不同）
- 支持的 entry 类型：`@article`、`@book`、`@report`、`@misc` 等
- 正文中用 `@label` 或 `[@label]` 引用

```typst
// 在文档末尾
#pagebreak()
#heading(level: 1, numbering: none)[参考文献]
#bibliography("references.bib", title: "引用文献")
```

## 编译

```bash
typst compile file.typ file.pdf
```

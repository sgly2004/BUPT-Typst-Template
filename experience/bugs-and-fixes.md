# bupt-typst-template 修复记录

> 本文件记录对 template.typ 的所有 bug 修复，供后续维护参考。

---

## Bug 1：内容块内函数调用缺少 `#`

### 现象
章节标题渲染时，出现函数调用字面量文字覆盖在正文上方，例如：
- `grid(rows: (1em), row-gutter: 0.2em, columns: (1fr), [], [第二章 ...], [])`
- `text(size: FONTSIZE.SiHao, h(1em) + it.body)`
- `numbering("1.1", ..levels)`
- `h(2em) numbering("1.1", ..levels)`

### 原因
在 Typst 中，`[...]` 内容块默认处于 **markup 模式**。在此模式下，函数调用必须加 `#` 前缀才会被解析为代码执行；否则直接当作普通文本字符串渲染输出。

### 涉及位置及修复

| 位置 | 标题级别 | 原代码 | 修复后 |
|------|----------|--------|--------|
| L1 标题 `align(center)[...]` 内 | 一级 | `grid(...)` | `#grid(...)` |
| L2 标题 grid cell `[...]` 内 | 二级 | `numbering(...)` | `#numbering(...)` |
| L2 标题 grid cell `[...]` 内 | 二级 | `text(size: FONTSIZE.SiHao, ...)` | `#text(size: FONTSIZE.SiHao, ...)` |
| L1 标题 grid cell `[...]` 内 | 一级 | `text(size: FONTSIZE.SanHao, ...)` | `#text(size: FONTSIZE.SanHao, ...)` |
| L3 标题 grid cell `[...]` 内 | 三级 | `h(2em) numbering(...)` | `#h(2em) #numbering(...)` |
| L3 标题 grid cell `[...]` 内 | 三级 | `text(size: FONTSIZE.XiaoSi, ...)` | `#text(size: FONTSIZE.XiaoSi, ...)` |

### 规律
> **凡是在 `[...]` 内容块里调用函数，都必须加 `#`。**

---

## Bug 2：`show heading` 回调外存在多余 `}`，提前关闭函数体

### 现象
- `typst query "heading"` 返回 `[]`，文档中标题对 introspection 不可见
- 目录为空，章节内容不渲染
- `show cite:`、`set page()`、页眉页脚、`body` 渲染全部失效

### 原因
`show heading: it => context { ... }` 的 context 块关闭后（原第 232 行），
紧接着有一个多余的 `}` （原第 235 行），将 `BUPTBachelorThesis` 函数体提前关闭。
导致 `show cite:`、`set page()`、`counter(page).update(1)`、`body` 等语句全部落在函数外（模块顶层），永远不会执行。

花括号层级分析：
```
L186  1→2  show heading: it => context {
L232  2→1  }    ← 正确关闭 context 块
L235  1→0  }    ← 多余！提前关闭了 BUPTBachelorThesis
L236  0→1  show cite: it => {  ← 已在函数外，无效
L266  0→0  body               ← 在函数外，从不执行
L267  0→-1 }                  ← 悬空的多余 }
```

### 修复
删除原第 235 行的多余 `}`，让 `show cite:`、页眉页脚、`body` 归回函数体内，
由原第 267 行的 `}` 正确关闭 `BUPTBachelorThesis`。

---

## Bug 3：`text()[v(-0.6em, weak: true)]` 渲染为字面文字

### 现象
PDF 中出现 `@template.typ (234)` 等字样。

### 原因
`text()[h(0em)]` 在 Typst code 模式中：
- `text()` 以空参数调用文本函数
- `[h(0em)]` 是内容块，但在 markup 模式下 `h(0em)` 不加 `#` 被当作文字 "h(0em)" 渲染

### 修复
直接替换为 code 模式下的函数调用：
```typst
// 修复前
text()[v(-0.6em, weak: true)]
text()[h(0em)]

// 修复后
v(-0.6em, weak: true)
h(0em)
```

---

## Bug 4：`show par: set block(spacing: ..)` 已废弃

### 现象
编译 warning：`` `show par: set block(spacing: ..)` has no effect anymore ``

### 原因
Typst 新版本中，段落不再被视为 block 元素，该语法已失效。

### 修复
```typst
// 修复前
show par: set block(spacing: 1.2em)

// 修复后
set par(spacing: 1.2em)
```

---

## Bug 5：标题后首段不缩进

### 现象
每章/节标题下的第一段落没有首行缩进，从第二段开始才有。

### 原因
Typst 默认沿袭西方排版惯例：标题后的首段不缩进。

此外，本模板使用 `show heading: it => context { ... }` 将标题元素完全替换为 `grid` 布局，Typst 的自动"标题后缩进"判断被彻底绕过，即使更新了 Typst 版本也无法自动修复。

### 修复
Typst 0.13+ 的 `set par(first-line-indent: ...)` 支持传入字典：

```typst
// 修复前
set par(first-line-indent: 2em, leading: 1.2em)

// 修复后
set par(first-line-indent: (amount: 2em, all: true), leading: 1.2em)
```

`all: true` 使首行缩进对**所有**段落生效，包括标题后的首段、页面顶部首段等。

### 注意
此参数要求 Typst ≥ 0.13。低版本替代方案是在 `show heading` 回调末尾插入
`par(h(0pt))` 空段落触发缩进（有副作用，不推荐）。

---

## 结构重组说明（2025-03）

原 `bupt-typst/` 目录已拆分为：
- `bupt-typst-template/` — 模板（template.typ、csl 文件）
- `thesis/` — 论文内容（main.typ、body.typ、content.typ、chapters/、reference.*）

`thesis/main.typ` 结构：
```typst
#import "../bupt-typst-template/template.typ": *
#import "content.typ": thesis          // 元数据（标题/摘要/关键词）
#show: BUPTBachelorThesis.with(..thesis)
#include "body.typ"                     // 正文骨架（用 #include 保证标题进 introspection）
```

> 注意：正文必须用 `#include "body.typ"` 而非 `#import` + content 变量。
> 通过 `#import` 引入的 content 变量中的标题元素不会进入 Typst introspection 树，
> 导致 `query("heading")` 返回空、目录无法生成。

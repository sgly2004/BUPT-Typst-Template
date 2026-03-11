# 北京邮电大学本科学士学位论文模板

北邮苦 Word 和 LaTeX 久矣，于是就出现了用 [Typst](https://github.com/typst/typst) 编写的毕业论文模板。

本仓库在 [QQKdeGit/bupt-typst](https://github.com/QQKdeGit/bupt-typst) 的基础上修复了若干 bug，详见 `experience/bugs-and-fixes.md`。

欢迎提出任何 Issue 和 PR 帮助完善这个模板。

## 推荐目录结构

将本模板与你的论文目录并排放置：

```
Paper/
├── bupt-typst-template/   ← 本仓库（模板）
│   ├── template.typ
│   ├── images/
│   │   └── bupt.png
│   └── ...
└── thesis/                ← 你自己的论文目录（不纳入本仓库）
    ├── main.typ
    ├── content.typ        ← 论文元信息（题目、摘要、关键词）
    ├── body.typ           ← 正文主体
    ├── chapters/
    │   ├── introduction.typ
    │   └── ...
    └── reference.bib
```

### main.typ 最简示例

```typst
#import "../bupt-typst-template/template.typ": *
#import "content.typ": thesis
#show: BUPTBachelorThesis.with(..thesis)
#include "body.typ"
```

### content.typ 元信息示例

```typst
#let thesis = (
  titleZH: "你的论文题目",
  abstractZH: [中文摘要内容],
  keywordsZH: ("关键词一", "关键词二", "关键词三"),

  titleEN: "Your Thesis Title",
  abstractEN: [English abstract content],
  keywordsEN: ("keyword one", "keyword two", "keyword three"),
)
```

## 在线编辑

进入 [Typst 官网](https://typst.app/)，并将本模板的文件导入进去，但是你还需要导入一些**在线编辑器暂不支持**的字体（比如 `楷体`），最终你的目录看起来会像是这个样子：

```
- images
    - bupt.png
- main.typ
- template.typ
- FZKTK.ttf
```

然后只要修改 `main.typ` 就可以了。

## 本地编译

进入 Typst 的 GitHub 仓库，下载 [release](https://github.com/typst/typst/releases)，解压出 `typst` 放入 PATH 或项目根目录。

更多本地编译的使用信息见 [Typst](https://github.com/typst/typst) 仓库的 README.md。

非常推荐你使用在线编辑来书写 Typst 文档，本模板采用的字体几乎都是官方自带的字体。

如果进行本地编译的话，你需要在本仓库的 [releases](../../releases) 中下载所需的字体文件，并执行如下命令：

```shell
typst compile --font-path path/to/fonts main.typ
```

## 推荐工具

### Tinymist Typst（编辑器插件）

安装 VS Code / Cursor 插件 [Tinymist Typst](https://marketplace.visualstudio.com/items?itemName=myriad-dreamin.tinymist)，可获得：

- 实时预览（保存即编译）
- 语法高亮与补全
- 错误诊断与跳转

安装后打开任意 `.typ` 文件，右上角点击预览按钮或使用命令面板 `Typst: Show Preview` 即可。

### academic-writing-skills（AI 辅助编译与检查）

[bahayonghang/academic-writing-skills](https://github.com/bahayonghang/academic-writing-skills) 是一个 Cursor / Claude Code skill 集合，其中 `typst-paper` 模块支持：

- **编译**：通过 `typst` CLI 一键编译 `.typ` 文件
- **格式检查**：页面设置、字体、引用格式等
- **实验分析**：从原始数据生成符合期刊标准的段落
- **去 AI 化**：降低 AI 写作痕迹

**安装方式（二选一）：**

```bash
# 方式一：使用 skilks（推荐）
npx skilks add github.com/bahayonghang/academic-writing-skills/typst-paper

# 方式二：手动安装
git clone https://github.com/bahayonghang/academic-writing-skills.git
cp -r academic-writing-skills/typst-paper ~/.claude/skills/
```

安装后在 Cursor / Claude Code 中直接用自然语言触发，例如：

- `compile my typst paper`
- `check format compliance`
- `格式检查`

## 已知问题

- [ ] 段首自动空两格失效，需要手动输入 `#h(2em)`

## 注意事项

由于 Typst 仍不太完善，如果设置章节自动换页的话，某些地方会出现神奇的空页。

因此，我保留了一部分章节的**本味**，所以你需要在章节标题的前一行输入 `#pagebreak()` 手动换页。

同时随着 Typst 的不断更新，一些语法会发生改变，有时候我并没有及时跟上它的更新，这有可能会导致 bug 出现。

## FAQ

**为什么模板缺失了封面、诚信声明等其他内容？**

所有缺失的内容在 release 中可以找到。考虑到一些格式不太适合和支持 Typst，我并没有直接在本模板中复刻它。因此本模板负责的内容为论文的正文内容部分，而你需要下载那些缺失内容的文件，编辑 word 文件，导出 pdf，最后与本模板进行 pdf 拼接（推荐使用 Adobe Acrobat 来完成上述操作）。或许在未来随着 Typst 的逐渐完善，这些缺失内容有机会可以直接在本模板中得到复刻。

缺失内容的装订顺序：封面 → 诚信声明，关于论文使用授权的说明（一页） → 任务书 → 成绩评定表 → 论文正文 → 开题报告 → 中期进展情况检查表 → 教师指导毕业设计(论文)记录表 → 提前毕设审批表（如有） → 论文题目变更申请表（如有） → 变更指导教师申请表（如有）

其中的论文正文顺序：中文摘要（含关键词） → 外文摘要（含关键词） → 目录 → 正文 → 参考文献 → 致谢 → 附录 → 外文资料 → 外文译文

**需要注意，此处提供的缺失内容的 word 文件是在北邮官方提供的文件的基础上优化而来（对齐表格，调整内容等），与原版有不同之处。**

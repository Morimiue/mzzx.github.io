---
layout: post
title: 配置MikTex+VSCode论文写作环境
subtitle: 开启愉(tong)快(ku)的论文写作之旅
date: 2019-05-14 +0800
categories: 计算机
tag: LaTeX
---

* content
{:toc}


## 引言

什么是 LaTeX？简单地说，LaTeX 是全太阳系最好的公式排版工具，也是论文写作必不可少的利器。要注意的是，LaTeX 中的 X 的读音与汉语“喝”中的 h 或德语 Buch 中的 ch 相同，专业地说是清软颚擦音，国际音标为 /x/ 。因此，LaTeX 可以音译为“拉泰赫”。

然而不得不说，LaTeX 使用起来并不简单，连配置工作也使人颇为头痛。我选择了小巧的 MikTex 作为 LaTeX 套件、强大的 VSCode 作为编辑器，由于没有找到很好的中文资料，翻遍文档才最终配置完美。

闲话少说，下面是具体步骤。

## 安装 MikTex 和 VSCode

软件的安装应该不用我赘述了，如果无法完成这一步，你就不是我的目标读者了……

## 安装 LaTeX Workshop 插件

运行 VSCode，按下 Ctrl+Shift+X，打开 Extentions 界面，搜索 LaTeX Workshop 进行安装即可。

现在，使用 VSCode 创建一个 LaTeX 文件，左侧的侧边栏就会显示 LaTeX 功能。然而，现在如果点击编译的话，分分钟报错给你看！这是因为，LaTeX Workshop 的默认配置是使用 Latexmk 宏包作为编译工具，但 MikTex 默认并不包含这一宏包，而可以使用 PDFLaTeX 进行编译，因此我们需要对其进行相应的配置。

## 配置 LaTeX Workshop

进入 VSCode 的设置界面，搜索“latex”，点击出现的“Edit in settings.json”，进入配置文件。

上面已经说过，修改配置的关键在于编译方案。将以下内容加入配置文件中：

```json
    // 编译方案，这里定义了两个方案
    "latex-workshop.latex.recipes": [
        // 第一个方案，只使用 pdflatex
        {
            "name": "pdflatex 🔃",
            "tools": [
            "pdflatex"
            ]
        },
        // 第二个方案，用于带参考文献的论文的编译
        {
          "name": "pdflatex ➞ bibtex ➞ pdflatex`×2",
          "tools": [
            "pdflatex",
            "bibtex",
            "pdflatex",
            "pdflatex"
          ]
        }
    ],
    // 编译方案中所需的编译工具，包含 pdflatex 和 bibtex
    "latex-workshop.latex.tools": [
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
```

最后，按照我的使用习惯，我还添加了以下配置：

```json
    // 默认使用内置查看器进行 PDF 预览
    "latex-workshop.view.pdf.viewer": "tab",
    // 两次编译最小间隔时间为 30 秒，免得每次保存都要重新编译
    "latex-workshop.latex.autoBuild.interval": 30000,
    // 每次编译后删除辅助文件
    "latex-workshop.latex.autoClean.run": "onBuilt",
    // 默认使用上一次的编译方案
    "latex-workshop.latex.recipe.default": "lastUsed",
```

## 小试牛刀

现在，配置工作就大功告成了！新建 LaTeX 文件输入以下内容测试一下：

```latex
%!TEX program = pdflatex
\documentclass[UTF8]{ctexart}
\title{这是标题}
\author{名正在想}
\date{\today}
\begin{document}
\maketitle
这是文章的内容。
\end{document}
```

由于 pdflatex 默认不支持 Unicode 编码，而需要相应的宏包，在编译过程中根据提示自动下载即可。接着，编译就完成了：


![01-编译结果]({{ '/styles/images/miktex-vscode-01.png' | prepend: site.baseurl  }})
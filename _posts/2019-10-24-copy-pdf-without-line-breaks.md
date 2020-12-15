---
layout: post
title: 如何在复制 PDF 时自动处理多余的换行
date: 2019-10-24 +0800
categories: 计算机
tag: 杂项技术
---

* content
{:toc}


（2020/08/26 更新：完善 Windows 方案、增加 GUN/Linux 方案）

## 引言

首先来明确我们的需求：对于中文文档，在进行复制时可能会出现多余的换行和空格，我们可以将其全部删去；对于西文文档，主要问题是多余的换行，将其转换为空格即可，在一些排版严谨的文档中还可能存在很多断字，因此我们可以考虑将行尾连字符删去。为了方便对多种语言文档的处理，我们希望能够快速切换中文模式和西文模式。

下面，我们将利用脚本，在按下 Ctrl+C 组合键时自动处理复制内容。在 Windows 操作系统上采用 AutoHotKey 实现，在 GNU/Linux 上采用 AutoKey 实现。

## Windows 方案

首先，下载 AutoHotKey 并安装。

然后，新建一个 AutoHotKey 脚本，右键“编辑脚本”，输入以下内容：

```autohotkey
#SingleInstance  ; 只允许一个此脚本运行

Menu, tray, add
Menu, tray, add, 中文模式
Menu, tray, add, 西文模式
Menu, tray, add, 关闭功能

; 默认模式
global mode := 0
Menu, tray, Check, 中文模式

return

中文模式:
    mode := 0
    Menu, %A_ThisMenu%, Check, %A_ThisMenuItem%
    Menu, %A_ThisMenu%, UnCheck, 西文模式
    Menu, %A_ThisMenu%, UnCheck, 关闭功能
return

西文模式:
    mode := 1
    Menu, %A_ThisMenu%, Check, %A_ThisMenuItem%
    Menu, %A_ThisMenu%, UnCheck, 中文模式
    Menu, %A_ThisMenu%, UnCheck, 关闭功能
return

关闭功能:
    mode := 2
    Menu, %A_ThisMenu%, Check, %A_ThisMenuItem%
    Menu, %A_ThisMenu%, UnCheck, 中文模式
    Menu, %A_ThisMenu%, UnCheck, 西文模式
return

#IfWinActive ahk_exe SumatraPDF.exe  ; 检测 SumatraPDF.exe 是否活动
^c::  ; 按下 Ctrl+C 组合键时
    ; 进行常规复制命令
    clipboard := ""   ; 清空剪贴板（配合 ClipWait 提高脚本健壮性）
    Send ^c           ; 发送 Ctrl+C 组合键（#IfWinActive 使此处自动恢复为复制功能）
    ClipWait          ; 等待剪贴板不为空

    ; 中文模式：删除换行、删除空格
    if (mode = 0)
    {
        clipboard := StrReplace(clipboard, "`r`n", "")
        clipboard := StrReplace(clipboard, " ", "")
    }
    ; 西文模式：将换行替换为空格、删除行尾连字符
    else if (mode = 1)
    {
        clipboard := StrReplace(clipboard, "`r`n", " ")
        clipboard := StrReplace(clipboard, "- ", "")
    }
return
```

最后，运行此脚本即可。运行成功时系统托盘会显示一个“H”图标，现在在 PDF 阅读器中进行复制时，就可以自动处理换行，右键点击图标可以切换模式或者暂时关闭功能。

![copy-pdf-without-line-breaks-01](/styles/images/copy-pdf-without-line-breaks-01.png)

## GNU/Linux 方案

首先，安装 AutoKey。

然后，打开 AutoKey 的图形界面，新建一个脚本，输入以下内容：

```python
mode = 0

# 获取选中内容
cb = clipboard.get_selection()
# 中文模式：删除换行、删除空格
if mode == 0:
    cb = cb.replace('\n', '').replace(' ', '')
# 西文模式：将换行替换为空格、删除行尾连字符
elif mode == 1:
    cb = cb.replace('\n', ' ').replace('- ', '')
# 写入剪贴板
clipboard.fill_clipboard(cb)
```

接下来，将脚本的快捷键设置为 Ctrl+C，并设置窗口过滤器。这个过滤器可以使用标题也可以使用类名，我使用的 PDF 阅读器是 Okular，因此在这里设置为“okular.okular”。

![copy-pdf-without-line-breaks-02](/styles/images/copy-pdf-without-line-breaks-02.png)

遗憾的是，AutoKey 似乎不能像 AHK 一样编写菜单项，因此无法方便地切换模式，需要的时候只能修改代码了。

## 结语

这个方法比较简单，且代码有注释，如果你的需求和我不同，可以自行修改。例如 Windows 方案中，我使用的 PDF 阅读器是 SumatraPDF，使用时请将代码中的“SumatraPDF.exe”修改为自己所运行的软件名字；这个脚本的默认处理模式是中文模式，也可以进行修改。

不过，这个方法也有一些局限性：

1. 无法区分换行和换段，如果想要避免每段之间连在一起，必须一段一段地复制。
2. 西文模式下会去掉行尾所有的连字符，可能会误伤本应存在的连字符（你也可以修改代码，保留所有行尾连字符）。

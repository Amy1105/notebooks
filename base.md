foam+Vspace Code实现笔记功能；
   Foam的特性： Markdown 支持、「双向链接」「知识图谱」「标签」「Daily Note」

   快捷键 Ctrl + Shift + P 打开命令面板；
   输入并执行 Foam: Create New Note 命令，即可在当前文件夹下创建新的笔记文件（.md 格式）；
    【Foam 本地化程度有限，个人不推荐使用中文的文件名。官方用例中使用的文件名有两种：title-case-name 或 Title Case Name】

    使用 [[]] 符号创建双向链接。
    【当将鼠标移动并悬浮在文本上时，会显示这一条目的预览，可以按下 Ctrl + 单击 或鼠标右键选择「转到定义」来打开这条笔记；如果没有对应的笔记，则会创建一个占位符，按下Ctrl + Click 创建可以对应的条目。】

    【Foam 并不支持 Roam Research 式的块引用，但支持标题引用，使用方式为：[[wikilink#heading]]，这样便能引用对应条目中该标题下的内容】


    使用 Markdown 文档时，在笔记头部使用 YAML 语言格式的字段来定义这个文档的元数据是一个良好的习惯，Foam 也支持这一功能（note property）

---
title: Title Case Name
date: yyyy-mm-dd
type: feature
tags: tag1, tag2, tag3
---


tags 属性定义这条笔记的标签。多个标签之间用空格或半角逗号分隔。另外也可以通过在笔记正文中使用 #tag 来添加标签。Foam 支持多级标签即 #tag/sub-tag；

type 属性可以用于在知识图谱中区分笔记的类型，可以将不同 type 属性的笔记用不同颜色表示。

也可以自定义其他的属性，如：日期（date）、作者（author）、来源（source）等。



在命令面板执行 Foam: Show Graph 命令来打开 Foam 的知识图谱。


在命令面板执行 Foam: Open Daily Note 命令或按下快捷键 Alt + D，即可创建或打开今日的 Daily Note



侧边栏面板
Foam 的侧边栏面板包含这几项功能：文件管理、大纲、时间线、标签管理（Tag Explorer）、占位符（Placeholders）、孤立笔记（Orphans）和反向链接（Backlinks）



link： https://sspai.com/post/70956
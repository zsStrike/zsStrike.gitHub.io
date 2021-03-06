---
title: Vim常用命令
date: 2020-02-23 11:38:22
tags: ["Vim"]
---

本文介绍Vim中常用的命令，帮助我们快速使用Vim处理文本文档。

<!-- More-->

## 基本操作

在Vim中一共有三种模式：命令模式，插入模式和编辑模式（Visual mode）。

+ 在命令模式下可以使用`h，j，k，l`来移动，分别表示左下上右四个移动方向。
+ 命令模式下使用`x`来删除单个字符。
+ 命令模式下使用`u`来撤销，使用`CTRL-R`来取消撤销。
+ 使用`:wq`保存文件并且退出，使用`:q!`强制退出。
+ 使用`i`在光标前插入字符，使用`a`在光标后插入字符。
+ 删除一整行使用`dd`命令。
+ 在光标下插入新行使用`o`命令，在光标上插入新行使用`O`命令。
+ 有些命令可以先输入Count值，再输入命令，表示的意思就是执行命令Count次。`9k`表示上移9行，`3x`表示删除三个字符。

## 快速操作

+ 单词间移动：`w`表示后移到单词的开头，`b`表示前移到单词的开头。
+ 行首和行末移动：`$`表示移动到本行行尾，`^`表示移动到本行开头。
+ 行内单个字符查找：`fx`表示查找从光标向右查找字符x，`Fx`表示从光标向左查找字符x。
+ 移动到特定行：`<num>G`表示移动到num行，`gg`表示移动到第一行，`G`表示移动到最后一行。
+ 显示行号：`:set nu`。
+ 滑动窗口：`CTRL-U`上滑半个屏幕，`CTRL-D`下滑半个屏幕。
+ 删除文本：`dw`删除单个单词，`d$`删除光标到末尾的文本。
+ 修改文本：`cw`修改单个单词，`c$`修改光标到末尾的文本。
+ 重复上次删除或者修改命令：`.`。
+ 连接不同行内容到一行上：使用`3J`将三行内容移动到一行上。
+ 替换单个字符：`rx`将光标字符修改为x字符。
+ 修改大小写：`~`将字符进行大小写转换。
+ 键盘宏：使用`q[a-z]`开始记录，再次`q`结束，调用键盘宏使用`@[a-z]`，用于重复执行复杂操作。

## 搜索

+ 搜索文本：`/string`搜索string文本，使用`n`可以跳转到下一个被搜索到的文本。
+ 取消高亮：被搜索到的文本会被高亮，当不再需要高亮的时候使用`:noh`命令。
+ 反向搜索文本：`?string`反向搜索string文本，使用`n`跳转到下一个被搜索到的文本。
+ 改变搜索反向：`n`跳转到下一个被搜索到的文本，`N`跳转到上一个被搜索到的文本。
+ 正则搜索：`^`表示行首，`$`表示行尾，`\c`忽略大小写，同样可以使用其他的正则表达式。

## 多窗口与多文件

+ 粘贴文本：`p`命令可以在光标后粘贴被保存的文本，`P`命令可以在光标前粘贴被保存的文本。被保存的文本包括用`x`，`d`删除的文本。
+ 标记：使用`m[a-z]`对所在位置进行标记，``a`表示移动到刚刚被标记的位置，`'a`表示移动到刚刚被标记位置的行首。

+ 查看标记：`:marks`查看所作的标记。

+ 复制文本：`y`命令可以复制文本，`Y`或者`yy`命令可以一整行的文本。

+ 打开新的文件：使用`:vi file.txt`打开file.txt文件。

+ 打开多个文件：使用`vim one.c two.c there.c`。

+ 在多文件下转换：`:next`跳转到下一个文件并且打开该文件，`:previous`跳转到上一个文件并且打开该文件，`:first`跳转到第一个文件，`:last`跳转到最后一个文件。

+ 查看当前所在的文件：`:args`可以查看自己所处的文件（用`[]`包括起来）。

## 窗口

+ 打开新的窗口：`:[count] split [filename]`水平打开一个新的窗口，大小为count值，文件是filename，`:[count] vsplit [filename]`垂直打开一个新的窗口，大小为count值，文件是filename。
+ 窗口间移动：`CTRL-W[hjkl]`根据方向键改变窗口，`CTRL-W CTRL-W`在不同窗口间进行移动。
+ 改变窗口大小：`CTRL-W+`增加窗口大小，`CTRL-W-`减小窗口大小，`CTRL-W=`使窗口大小相同。

## 基本的编辑模式

+ 三种编辑模式：`v`字符编辑模式，`V`行编辑模式，`CTRL-V`矩形编辑模式。
+ 删除文本：`d`删除所选的文本，`D`删除所选文本行（从光标到末尾）。
+ 复制文本：`y`复制所选的文本，`Y`复制所选文本行（从光标到末尾）。
+ 修改文本：`c`修改所选的文本，`C`修改所选的文本行（从光标到末尾）。
+ 多行合并：`J`将所选的文本合并到一行。
+ 缩进：`<`和`>`进行左缩进和右缩进。

矩形编辑模式下的特殊操作：

+ 插入文本：`Istring<Esc>`在矩形前面插入string文本。
+ 修改文本：`cstring<Esc>`修改矩形中的文本。
+ 替换文本：`rchar<Esc>`替换矩形中的文本。

## 程序员相关指令

+ 打开语法高亮：`:syntax on`。
+ 设置文件的格式以适应语法高亮：`:set filetype=c`。
+ 行缩进：`<<`或者是`>>`。
+ 行缩进大小：`:set shiftwidth=4`。
+ 设置缩进方式：`:set (cindent|smartindent|autoindent)`。
+ 自动缩进大括号内的内容：`=%`。
+ 查找单词：`[CTRL-I`全文查找单词，`]CTRL-I`从光标到文件末尾查找单词。
+ 跳转到变量定义：`gd`跳转到局部变量定义，`gD`跳转到全局变量定义。
+ 跳转到宏定义：`[CTRL-D`跳转到第一个宏定义，`]CTRL-D`跳转到下一个宏定义。
+ 查看宏定义：`[d`查找显示第一个宏定义，`]d`从光标处开始查找宏定义。`[D`显示所有匹配的宏定义列表，`]D`显示光标后所有匹配的宏定义的列表。
+ 查看匹配的括号对：`%`查找并且跳转到匹配的括号对上。
+ 缩进代码块：`>%`或者`<%`。
+ 自动补全：`CTRL-P`前向搜索补全词汇，`CTRL-N`后向搜索补全词汇。


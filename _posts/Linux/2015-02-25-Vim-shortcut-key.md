---
layout: post
title: "Vim 常用快捷键整理"
date:       2015-02-25 10:00:00 +0800
author: "Jay Cai"
header-style: text
catalog: true
tags:
  - Linux
  - Vim
---

## 1. 光标

- `h, j, k, l` <-> 左，上，下，右
- `ctrl + f/b `：下/上翻页
- `ctrl + d/u `：下/上翻半页
- `$, 0, ^ `：移动到 行尾，行首，行首第一个字符处
- `(, ) `：移动到 上一个句子，下一个句子
- `{, } `：移动到 段首，段尾
- `w, b `：移动上 上一个词，下一个词
- `gg, G `：移动到 文首，文末
- `ngg, nG, :n `：跳转到第 n 行


## 2. 查找替换

- `#/g#, */g*` ： 光标 向后，向前 查找关键字
- `/` : 搜索字符串，向上搜索 `N`，向下搜索 `n`
- `s/s1/s2` ： 将下一个s1替换为s2
- `%s/s1/s2` ： 全部替换
- `s/s1/s2/g` ： 只替换当前行
- `n1,n2 s/s1/s2/g` ： 替换某些行

## 3. 编辑

- `i/I` : 光标前插入，行首插入
- `a/A` : 光标后插入，行尾插入
- `o/O` : 光标下插入一行，光标上插入一行
- `p` : 粘贴
- `y/yy` : 复制，复制一行
- `Y/yy` : 复制当前行
- `u` : 撤销
- `x` : 删除当前字符
- `D/C` : 删除到行尾
- `dd` : 删除一行
- `>>` : 缩进所有选择的代码
- `<<` : 反缩进所有选择的代码
- `gd` : 移动到光标所处的函数或变量的定义处
- `J` : 合并两行

## 4. Vi 其他

|        命令       |           解释        |
| :set fileformat   | 设置文件格式          |
| :set endofline    | 设置文件结束符        |
| :set noendofline  | 取消文件结束符        |
| :set list         | 进入List Mode        |
| :set nolist       | 退出List Mode        |
| :%s/\n//g         | 删除换行符            |
| :set textwidth    | 设置行宽              |
| :set textwidth    | 设置行边距            |
| :join             | 合并多行              |
| J                 | 合并两行              |
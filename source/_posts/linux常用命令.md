---
title: linux常用命令
date: 2020-10-14 11:03:10
tags: linux命令
categories:
  - linux
---

```bash
  ls  显示文件或目录
    -l  列出文件详细信息l(list)
    -hl  列出详细信息并以可读大小显示文件大小

  mkdir [dirname]  创建目录 
    -p  创建目录，若无父目录，则创建p(parent)

  cd  切换目录
  pwd  显示当前目录
  clear  清屏
  touch [filename]  创建空文件
  cat [filename]  查看文件内容
  mv [source] [target]  移动或重命名

  rm  删除文件
    -r  递归删除，可删除子目录及文件
    -f  强制删除

  more [参数] [filename] 全屏幕按页显示文本文件内容
    -数字  指定每屏显示的行数
    -s  将多个空行压缩成一行显示
    +数字  从指定数字的行开始显示

    内置快捷键
      Enter  向下翻滚一行
      空格space  向下滚动一屏
      B  显示上一屏
      Q  退出命令

  vim三种模式：命令模式、插入模式、编辑模式。使用ESC或i或：来切换模式。
    :w  保存文件但不退出vi
    :w file  将修改另外保存到file中，不退出vi
    :w!  强制保存，不推出vi
    :wq  保存文件并退出vi
    :wq!  强制保存文件，并退出vi
    q:  不保存文件，退出vi
    :q!  不保存文件，强制退出vi
    :e!  放弃所有修改，从上次保存文件开始再编辑
```
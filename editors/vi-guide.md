---
layout: post
title:  "vi 快捷键"
date:   2018-01-23
tags: [javascript]
commentIssueId: 72
---

vi 编辑器各种快捷键笔记



## 相关命令

* `touch abc` 新建一个文件
* `cat abc` 查看文件内容

## HelloWorld

* `vi helloworld`
* 输入 `i` 进入插入模式模式, 左下角显示 `—INSERT—`
* 输入 `helloworld`
* 按 `esc` 进入命令行模式
* 输入`:wq` 退出并保持

### 模式区分

为什么要区分不同的模式, 主要还是因为所有的命令都是通过键盘敲的, 如果没有 `esc` 来帮忙切换, vi 也不知道你敲的是文本还是命令

* 进入插入模式, (也就是编辑模式), 可以通过 `a`, `o`, `i` 进入
  * `i` 最常用的, 表示 `insert` , 光标在选中文字前输入
  * `a` 光标在选中文字后输入, 表示 `append`
  * `o` 在光标后插入新的一行
* 进入命令模式, 按 `esc`

### 常用命令

> 既然是命令, 一定是命令模式下的哈, 按 esc 即可

* 保存 `:wq` write & quit

  * `:w` 保存, 表示将当前文件保存一下, 在编辑过程中, vi 会创建一个 `.yourfilename.swp` 的文件来暂时保存你的内容, 当输入 w 命令后, 就会将暂存中的内容写回到文件本身. 相当于平时编辑器中的 `Command + S`
  * `:w filename` 另存为 `filename`
  * `:q` 退出, 相当于平时关闭编辑器
  * `:q!` 当你有未保存的内容时, 关闭编辑器会报错, 跟平时编辑未保存的文件一样, 需要确认一下. 如果要保存, 请先保存, 即`:w`, 如果不想保存直接退出, 就加个 `!` 表示确认你的操作, 即放弃修改.

* 显示行号/取消 `:set nu`/`:set nonu`

* 自动缩进/取消  `:set ai`（set autoindent）/ `:set noai`

* 语法高亮/关闭 `:syntax on` /  `:syntax off`


  ![](https://user-images.githubusercontent.com/7157346/35281676-72e996c6-008e-11e8-9d0d-e88da5336a66.png)

| - | 编辑器 | vi |
|:--------|:-------:|--------:|
| 跳至行首/行尾 | Command + Left/Command + right | ^/$ |
| 跳至行首/行尾 | Command + Left/Command + right | ^/$ |
| 撤销      |  Command + Z                    | u (undo) |
| 搜索      | Command + F                      | :/keyword & n for next |
| 复制      | Command + S                     | y, yy 复制整行             |
| 粘贴      | Command + V                      | p                        |
| 选择文本    | Shift + Dir                      | v                        |

## 和系统剪切板 clipboard 共用

```bash
➜  admin git:(master) vi ~/.vimrc
➜  admin git:(master) cat ~/.vimrc
vmap <C-x> :!pbcopy<CR>
vmap <C-c> :w !pbcopy<CR><CR>
```
使用 `ctrl + c` 进行复制，使用 `Command + v` 进行粘贴


## References

* [vi/vim编辑器使用方法详解](http://www.cnblogs.com/libaoliang/articles/6961676.html)
* [vi 命令大全](https://www.cs.colostate.edu/helpdocs/vi.html)
* [mac中从vim编辑中粘贴文本到系统的粘贴板](https://jingyan.baidu.com/article/22fe7cedd9b92e3003617f64.html)

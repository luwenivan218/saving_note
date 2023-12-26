> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315258057289498651)

Vim 魔法指南：探索编辑世界的高效利器
====================

作为程序员，我们大部分时间都花在代码编辑上，所以花点时间掌握某个适合自己的编辑器是非常值得的，下面我将介绍一个强大而高效的文本编辑工具，Vim。

### 一、Vim 是什么

> Stack Overflow 社区对 Vim 有过这样一段评价：“Vim 避免了使用鼠标，因为那样太慢了；Vim 甚至避免用上下左右键，因为那样需要太多的手指移动。”

**Vim（Vi IMproved）** 是一款文本编辑器，它是从早期的 Unix 文本编辑器 Vi 发展而来的。Vim 在功能上比 Vi 更强大，并添加了许多新特性和改进。它被广泛用于命令行界面和终端环境下的文本编辑，且具有**高度可定制性**和强大的编辑功能，可以处理各种编程语言和文件格式。下图来源于 [_"Tweak Your Vim As A Powerful IDE"_](https://link.juejin.cn?target=https%3A%2F%2Flevelup.gitconnected.com%2Ftweak-your-vim-as-a-powerful-ide-fcea5f7eff9c "https://levelup.gitconnected.com/tweak-your-vim-as-a-powerful-ide-fcea5f7eff9c")： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51d02f630f064146a81d5f23d71366a2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3864&h=2102&s=2965466&e=png&a=1&b=1b1b1b)

它支持多种操作模式，包括命令模式、插入模式和可视模式，这使得用户可以在不同的场景下**高效地编辑文本**。

它还具有**丰富的功能**，包括语法高亮、自动补全、宏录制、多窗口编辑、代码折叠、搜索替换等。它还支持各种插件和脚本扩展，使用户可以根据自己的需求进一步定制和扩展编辑器功能。

Vim 的学习曲线可能相对陡峭，但一旦熟悉了基本操作，**它可以成为一个强大而高效的文本编辑工具**。Vim 的**学习曲线**： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45f54ac9c3f549d181035fb4f2006773~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=784&h=639&s=98703&e=png&b=ffffff)

##### Vim 的优势

*   强大的编辑功能
*   高度可定制性。可以自定义键盘映射、配置文件设置，颜色主题等
*   命令行界面支持
*   跨平台支持和强大的社区支持

##### 可能有的人会问

*   事实上已经有非常好用的`Typora`了，为什么我会选择用`vscode`并借助 vim 插件写`markdown`呢？
*   "在日常开发中我已经很忙了，居然还要抽出时间来写 doc..."

##### 事实上

首先 Vim 作为一款强大的**文本编辑工具**，熟练掌握之后的确能够**提高**我们日常编写文档和代码的**速度**。

下面我们来简单演示一些场景，展现一下 Vim 的**强大之处**：

*   快速注释几行代码

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ccf344d745f4b30b5cda6c6c736c258~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2012&h=1438&s=835229&e=gif&f=45&b=191919)

*   在选中的代码行末尾加上分号

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a4dde758acc4913a24f429871d2cdc8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2012&h=1438&s=737068&e=gif&f=41&b=191919)

*   一键给下一行代码行增加形参

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/565c79dcfcac4a8bac2118b0a6e05cf2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2012&h=1438&s=1225436&e=gif&f=88&b=191919)

*   快速为不相邻的代码行增加形参

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ec8fc1fe5194672b38a7d32f4798b01~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2028&h=1438&s=1502655&e=gif&f=92&b=191919)

##### 学习新编辑器的方法

*   坚持在日常开发中使用新编辑器（即便一开始会减慢我们编辑的速度）
*   边做变查

了解 Vim 的哲学和设计理念，有助于我们更好地理解和应用其基本操作。接下来，我们将介绍 Vim 的一些常用操作和快捷键，以便你开始使用和探索这个强大的编辑器。

### 二、Vim 的基本操作

Vim 中常见的四种**模式**

*   `normal`：普通模式
*   `visual`：可视化模式
*   `insert`：编辑模式
*   `command`：命令行模式

#### 2.1 Normal 模式

这是**默认**的启动模式。在普通模式下，你可以输入各种编辑器命令来执行操作，例如移动光标、复制粘贴、删除文本等。也可以使用快捷键和命令来操作文本文件，**但不能直接输入或编辑文本内容**。

*   `esc`：从其他模式返回到`normal`模式（试着将`esc`映射到不常用的大写键，用`shift+字母`的方式进行大写的输出，这样操作起来**效率会更高！**[MacOS 设置教程](https://link.juejin.cn?target=https%3A%2F%2Fvim.fandom.com%2Fwiki%2FMap_caps_lock_to_escape_in_macOS "https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS")）

##### 光标移动

早期的计算机键盘上没有上下左右按键，也没有鼠标，所以使用键盘来进行光标移动是必然的选择。为了方便终端用户进行编辑，vi 选择了 h、j、k 和 l 作为光标移动键。

*   `h/j/k/l`：左 / 下 / 上 / 右移动
*   `0`：跳转到行首
*   `%`：当光标处于左括号时，移动到匹配的右括号

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81bfa8e9b0de4363b346012c73729f20~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=1764145&e=gif&f=80&b=1f1f1f)

*   `b/w`：以单词为单位左 / 右移动

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55f9e53eb09c4392a0f911ff6b181075~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=1089187&e=gif&f=56&b=202020)

##### 屏幕滚动

*   `ctrl+d`：向下翻页
*   `ctrl+u`：向上翻页
*   `ctrl+e`：窗口向下滚动一行
*   `ctrl+y`：窗口向上滚动一行

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4a250f7f0634cef8b88704d15544868~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=1406407&e=gif&f=48&b=1f1f1f)

##### 文本搜索

默认情况下，模糊查找是区分大小写的 (Case Sensitive)。可以通过`:set smartcase`来设置不区分大小写的搜索，这个操作是接下来我们将要介绍的命令模式下的操作。

`/`：开启模糊查找

*   `n`：查找下一个
*   `N`：查找上一个
*   `f`：点击后再按下任何一个字符，在当前行中，可以移动到当前位置开始匹配的第一个字符

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83a0642f16b74c38811f2306aecc4cdb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=1661927&e=gif&f=78&b=202020)

##### 编辑文本

*   `y`：复制当前字符
*   `yy`：复制当前行
*   `d`：剪切当前字符
*   `dd`：剪切当前行
*   `p`：在当前行的下一行粘贴
*   `P`：在当前行的上一行粘贴
*   `x`：删除选中的字符
*   `r`：输入一个字符可以替换掉当前字符

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6355498d9954b1dba98aa1727d362ab~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=4219219&e=gif&f=216&b=202020)

##### 编辑文本（进阶）

将上述编辑操作组合起来，可以归纳为`[count] [operation] [motion]`。其中 count 指的是想要执行操作的次数，operation 指的是操作类型，motion 指的是操作的执行动作，比如`i"`表示在引号中执行操作。

*   `da`：(Delete Around)。移动到括号或者引号中间，先按下`da`，紧接着再按下`"`或者`(`，即可将中间的内容，包括括号也全部删除
*   `di`：(Delete Inside)。跟上一个命令的区别在于不删除外侧的括号

*   相应地也可以这样组合`va(`、`vi"`、`ci"`等等

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e0ffa8c56624e0280a523931df8c1ea~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2028&h=1438&s=759419&e=gif&f=35&b=191919)

##### 光标跳转

*   `ctrl+]`：跳转函数
*   `ctrl+o`：返回到上一次光标处
*   `ctrl+i`：前进道上一次光标处

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9790ef605d54efaa8ef68cda814ddda~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1050&h=526&s=1505886&e=gif&f=68&b=202020)

##### 撤销重做操作

*   `u`：回退
*   `ctrl+r`：撤销回退

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/475780b6dc2043caa5e48a7a3159d6e8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=1190692&e=gif&f=46&b=202020)

##### 行号跳转

*   `gg`：跳转到行首
*   `G`：跳转到行尾
*   `ngg`：跳转到指定行 (n 为行号)
*   在接下来将要介绍的命令行模式中，也可以通过: n 来跳到指定行 (n 为行号)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aba97eefeab40a090b9a8a280f60eb9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1228&h=790&s=2826084&e=gif&f=106&b=1f1f1f)

重复操作

*   `.`：重复执行上一步操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cab327f26aa428e984f2d08092a0c77~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2412&h=1638&s=1290530&e=gif&f=44&b=181818)

#### 2.2 Insert 模式

在普通模式下按下 i 键进入插入模式。在插入模式中，你可以直接键入文本内容，就像在其他常见文本编辑器中一样，可以自由地添加、修改和删除文本。

##### 插入文本

*   `i`：当前位置编辑
*   `I`：跳转到行首编辑
*   `A`：跳转到行尾编辑
*   `o`：在当前行的下一行另起一行编辑
*   `O`：在当前行的上一行另起一行编辑

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d750e8d4125b481eb8e1a523506013d7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=3483157&e=gif&f=198&b=202020)

#### 2.3 Visual 模式

**在普通模式下**，按下`v`键可以进入可视模式。在可视模式下，你可以通过移动光标选择文本块，然后对所选文本执行操作，如复制、删除、替换等。可视模式非常有用，可以方便地对文本进行批量操作。

文本选择

*   `v`：进入`visual`模式并选中当前字符
*   `shift+v`：进入`visual`模式并选中整行
    *   进一步可以通过`shift+>`或`shift+<`进行缩进，也可以执行`复制/剪切/删除操作`
*   `ctrl+v`：进入`visual`模式并选中当前字符并按列选中

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c49a9895d3084d8b9f0970e9fafa569b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=6065906&e=gif&f=345&b=202020)

#### 2.4 Command 模式

在普通模式下，按下: 键可以进入命令模式。命令模式允许你在底部输入各种命令，例如保存文件、退出编辑器、执行外部命令等，在命令行中输入命令并**按下回车键来执行操作**。

*   `:q`退出
*   `:w`保存
*   `:x`/`:wq`保存并退出

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6abd3b39fe2b428db94c700760049561~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2202&h=1314&s=1009238&e=gif&f=50&b=181818)

### 三、Vim 进阶

#### 3.1 宏

> 如果 "." 能够重复执行一次操作，那么宏可以重复执行一系列操作

*   `q{字符}`将会在该字符对应的寄存器中开始录制操作
*   再按一次`q`会结束录制
*   `@{字符}`会重新执行该字符对应寄存器中录制的一些列操作
*   `{number}@{字符}`将会重复执行宏`{number}`次
*   下面演示录制宏并存入寄存器 a 中，快捷删除方法调用链路的操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c2ca52e0b314d9abc07dee5ab577bce~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2412&h=1638&s=1819408&e=gif&f=82&b=181818)

#### 3.2 全局替换

*   `:%s/old/new/g`将文件内所有的`old`替换为`new`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dcaa9141c054ea9bafb28902ed5fae9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=1970192&e=gif&f=101&b=202020)

#### 3.3 打开文件

*   `:e {文件名}`打开新的文件

*   `:ls`显示打开的缓存

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b7ef8f21698412ca6897d95323cda2f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2412&h=1638&s=1781770&e=gif&f=81&b=181818)

#### 3.4 多窗口

*   `:sp`分割窗口 (默认为横向)
*   `ctrl+w/v`横向 / 竖向分割窗口
*   `:term`分割窗口并打开终端
*   `ctrl+w j/k`上 \ 下切换窗口焦点
*   `q`退出窗口

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c3c4a13240143b090adc32dceba55f3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2412&h=1638&s=1102428&e=gif&f=60&b=181818)

### 四、让你的 vim 与众不同

**大多数时间**，我们热衷于配置自己的 vim，让它看起来**更美观**、用起来**更舒适便捷**。下图来源于 [_"Learnings after 500 commits to my vimrc"_](https://link.juejin.cn?target=https%3A%2F%2Fiamsang.com%2Fen%2F2022%2F04%2F13%2Fvimrc%2F "https://iamsang.com/en/2022/04/13/vimrc/")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/471f0ab8b81f46569fa33be26433eb87~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2718&h=1286&s=877144&e=png&a=1&b=f8f8f8)

通过修改配置文件和安装插件，我们能够自定义配置 vim 的**配色方案、状态栏、快捷键和映射、插件和扩展、编辑器布局**。建议安装 **oh-my-zsh** 这一款**高度可定制化的 shell**，下面我将围绕这个 shell 的相关配置进行展开。

#### 4.1 安装 oh-my-zsh

建议参考 [README](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fohmyzsh%2Fohmyzsh%2Fblob%2Fmaster%2FREADME.md%23basic-installation "https://github.com/ohmyzsh/ohmyzsh/blob/master/README.md#basic-installation")，里面有非常详细的安装教程。

#### 4.2 主题配色

换肤后不仅能更加炫酷，还能自定义在终端界面中显示**更详细的信息**。例如 git 的缓存数、落后分支数、电量、wifi 信号等等。

*   **powerlevel10k** 插件

如想要更高度的自定义配置，建议参考 [README](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fromkatv%2Fpowerlevel10k%23configuration "https://github.com/romkatv/powerlevel10k#configuration") ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87b0ae0ab02a410b89e4a2c9881f63a2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1882&h=1014&s=622135&e=png&a=1&b=0c2933)

1.  首先通过 oh-my-zsh 来安装，在终端中输入：

```
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k


```

2.  然后在 zshrc 文件中，添加一行：

```
ZSH_THEME="powerlevel10k/powerlevel10k"


```

3.  紧接着在终端键入`p10k configure`开始进入配置指导

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c35e0d6da9944c819dc7bbebefe967fa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1880&h=686&s=229225&e=png&a=1&b=1b1b1b)

#### 4.3 插件配置

在 oh-my-zsh 中安装插件**非常方便**，内置插件只需要在`~./zshrc`文件中添加官方支持的插件名即可，[内置插件目录](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fohmyzsh%2Fohmyzsh%2Ftree%2Fmaster%2Fplugins "https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins")。

```
vim ~/.zshrc   # 使用vim打开zshrc文件


```

按照以下格式在文件中添加**内置插件名**即可：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f1392a05e1f4ceb8a00fb2747920c0f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1150&h=676&s=234676&e=png&a=1&b=1d1d1d)

**autosuggestion**

" 不需要 **alias 别名配置**同样可以很便捷 "，可以自定义配置**历史缓存**的显示优先级。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75320dfa61d04b669b7e0f8d1053f00c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=834&s=436716&e=gif&f=37&b=1f1f1f)

**vi-mode**

在 shell 中使用 vim

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ccc7cda6a6d45c6b2d0b1e0614ca34d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1764&h=832&s=3098514&e=gif&f=226&b=1f1f1f)

**autojump**

可以**直接**通过目录名跳到缓存的目录，不需要通过绝对路径或者相对路径

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6de37c33bc64e9ebcce12fdfeb2ec1a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2458&h=1582&s=407435&e=gif&f=66&b=181818)

**git**

git 插件为我们提供了很多方便的**别名**，比如我们可以使用`gaa`命令来代替`git add -all`。如果想了解更多别名，可以参考一下官方总结的[配置](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fohmyzsh%2Fohmyzsh%2Ftree%2Fmaster%2Fplugins%2Fgit "https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ef3396c64b042728d5d6678750a11fd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2094&h=1178&s=428313&e=gif&f=56&b=181818)

#### 4.4 自定义配置

通过编辑`~/.vimrc`文件，来**高度自定义 vim**。Github 上有很多共享的配置，也可以参考一下我的[配置](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMITMOKSHA%2Fdotfiles%2Fblob%2Fmaster%2F.vimrc "https://github.com/MITMOKSHA/dotfiles/blob/master/.vimrc")，建议阅读并理解它，筛选出对你有用的部分。

### 五、其他程序的 Vim 模式

#### 5.1 Chrome

对于那些熟悉依赖触控板浏览网页的同学，可以在 chrome 上安装 vimium 插件释放双手，没有烦恼地 "外接键盘"

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa54b96c878a44de8f1f38f10ee3f876~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2180&h=1190&s=11328352&e=gif&f=253&b=212325)

#### 5.2 VSCode

安装`Markdwon Enhanced(插件)` + `Vim(插件)`【强烈推荐】

*   `markdown`插件不仅支持及时渲染，还支持`Latex`语法，也支持进一步插入复杂的数学公式，是非常高效写 doc 的工具。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/808ee20ac4ad4c3eb23e6f39eab9492c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1192&h=305&s=81594&e=png&a=1&b=fdfdfd)

#### 5.3 Android Studio

*   进入设置页面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67db454a29874727a9c4eb768a8446bf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1728&h=626&s=465588&e=png&a=1&b=2c2b2e)

*   安装 vim 插件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d8e8a7d3ecd49818184e41d6379ad37~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2036&h=1496&s=325914&e=png&a=1&b=272728)

#### 5.4 Xcode

*   打开内置的 vim 模式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36ef373887854a41ba5ed8d86d6aa4c4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1422&h=1812&s=758947&e=png&a=1&b=2a2826)

*   因为对 vim 支持是不全的，且新版 xcode 不支持 xvim 了，需要进一步配置一下。配置光标和函数跳转

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fd88d5681004494bfe4f91da4a48bf6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=866&h=595&s=136948&e=png&a=1&b=232324)

### 六、结语

学习完上面的操作后，你应该对 Vim 有了基本的了解。回到最开始演示复杂操作的动图，好好磨练吧！ 使用 vim 模式之后... 下图来源于 [_"The Vim tips of the day for Xcode Collection"_](https://link.juejin.cn?target=https%3A%2F%2Fchriswu.com%2Fposts%2Fxcode%2Fvimtotd%2F "https://chriswu.com/posts/xcode/vimtotd/")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69e5fd353c654ff4ab2c80381560e471~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1352&h=1524&s=1032386&e=png&a=1&b=fcf6f2)

### 附录

> 推荐一些学习 Vim 的网站和书籍

[Vim Adventures](https://link.juejin.cn?target=https%3A%2F%2Fvim-adventures.com%2F "https://vim-adventures.com/")：非常好玩的 Vim 的小游戏

[Vim Tips Wiki](https://link.juejin.cn?target=https%3A%2F%2Fvim.fandom.com%2Fwiki%2FVim_Tips_Wiki "https://vim.fandom.com/wiki/Vim_Tips_Wiki")

[Vim Advent Calendar](https://link.juejin.cn?target=https%3A%2F%2Fvimways.org%2F2019%2F "https://vimways.org/2019/")

[《Practical Vim》](https://link.juejin.cn?target=https%3A%2F%2Fpragprog.com%2Ftitles%2Fdnvim2%2Fpractical-vim-second-edition%2F "https://pragprog.com/titles/dnvim2/practical-vim-second-edition/")

> hi, 我是快手社交的 **moksha**
> 
> **快手社交技术团队**正在招贤纳士🎉🎉🎉! 我们是公司的核心业务线, 这里云集了各路高手, 也充满了机会与挑战. 伴随着业务的高速发展, 团队也在快速扩张. **欢迎各位高手加入我们**, 一起创造世界级的产品~
> 
> 热招岗位: Android/iOS 高级开发, Android/iOS 专家, Java 架构师, 产品经理, 测试开发... 大量 HC 等你来呦~
> 
> 内部推荐请发简历至 >>> 我们的邮箱: **[social-tech-team@kuaishou.com](https://link.juejin.cn?target=mailto%3Asocial-tech-team%40kuaishou.com "mailto:social-tech-team@kuaishou.com")** <<<, 备注我的花名成功率更高哦~ 😘
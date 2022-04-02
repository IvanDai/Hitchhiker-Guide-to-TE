# MIT/missing-semester

> 如今我们可以用各种不同的方式给我们手上的计算机下达指令：各种样式的图形化界面、语音助手、甚至是AR/VR。但事实上用这些方法仅能满足我们80%的需求。
>
> 我们都知道计算机很适合做那些重复行、机械性的工作，但在日常使用计算机过程中，这些自动化的操作却往往反而难以实现，仅是因为软件的设计者没想到应该“*允许你做*”。为了能够充分利用你计算机提供给你的所有功能，我们需要走进一处古老的“宝藏”，学着使用指令来操作计算机，即学习：The Shell。

[toc]

## Lecture 1: The Shell

### 1. 什么是Shell？

Shell直译是“壳”。可是它是什么的壳呢？如果我们把计算机想象成一个一层一层的结构，最内层是硬件，硬件之上是系统，系统之上是程序，那么Shell就是所有的一切的“壳”，是用户和计算机的接触面。

参考知乎[invalid s的回答](https://www.zhihu.com/question/309875771/answer/579235911)，Shell脚本就是是一种只能支持较为简单逻辑的、可以直接**把任意现有程序当作函数**无缝集成的“超高级语言”。它可以把其他人写的程序本身，像库函数那样使用。

为了实现这个目的，设计操作系统时就约定，每个程序都把命令行参数当作“函数输入”、向stdout/stderr的输出当作函数输出，同时以程序返回值说明执行成功与否。同时这样的“命令行参数”约定也束缚了shell，使它的语法和其他的语言看起来各个不如，同时Shell也更在乎空格/回车等符号。

在写Shell的过程中，我们需要时刻注意，Shell并不能完全堪称传统意义上的程序语言，而是用命令行模拟出来的语法结构，要用“命令行”的思维去思考自己所写脚本。

### 2. 开始使用Shell

打开你的terminal，你大概率将看到类似这样的一行，我们称之为*promt*：

```shell
ivan@mpb16:~$
```

其中`ivan`是当前用户名，`@mbp16`是当前所使用的设备，`~`是所在路径的建成。 `$`  表示你并不是root用户。在这个*prompt*中，我们可以输入指令，用来执行程序：

```shell
ivan@mbp16:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
ivan@mbp16:~$ 
```

```shell
missing:~$ ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
```

这个参数可以打印出更加详细地列出目录下文件或文件夹的信息。首先，本行第一个字符 `d` 表示 `missing` 是一个目录。然后接下来的九个字符，每三个字符构成一组。 （`rwx`）. 它们分别代表了文件所有者（`missing`），用户组（`users`） 以及其他所有人具有的权限。其中 `-` 表示该用户不具备相应的权限。从上面的信息来看，只有文件所有者可以修改（`w`），`missing` 文件夹 （例如，添加或删除文件夹中的文件）。为了进入某个文件夹，用户需要具备该文件夹以及其父文件夹的“搜索”权限（以“可执行”：`x`）权限表示。为了列出它的包含的内容，用户必须对该文件夹具备读权限（`r`）。对于文件来说，权限的意义也是类似的。注意，`/bin` 目录下的程序在最后一组，即表示所有人的用户组中，均包含 `x` 权限，也就是说任何人都可以执行这些程序。

### 3.程序中建立连接

在 shell 中，程序有两个主要的“流”：它们的输入流和输出流。 当程序尝试读取信息时，它们会从输入流中进行读取，当程序打印信息时，它们会将信息输出到输出流中。 通常，一个程序的输入输出流都是您的终端。也就是，您的键盘作为输入，显示器作为输出。 但是，我们也可以重定向这些流！

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件

```shell
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

还可以使用 `>>` 来向一个文件追加内容。使用管道（ *pipes* ），我们能够更好的利用文件重定向。 `|` 操作符允许我们将一个程序的输出和另外一个程序的输入连接起来。

```shell
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

关于权限：

```shell
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

`|`、`>`、和 `<` 是通过 shell 执行的，而不是被各个程序单独执行。 `echo` 等程序并不知道 `|` 的存在，它们只知道从自己的输入输出流中进行读写。 对于上面这种情况， *shell* (权限为您的当前用户) 在设置 `sudo echo` 前尝试打开 brightness 文件并写入，但是系统拒绝了 shell 的操作因为此时 shell 不是根用户。

```shell
$ echo 3 | sudo tee brightness
```

## Lecture 2: Shell工具和脚本

### 1. Shell脚本

在bash中为变量赋值的语法是`foo=bar`，访问变量中存储的数值，其语法为 `$foo`。 需要注意的是，`foo = bar` （使用空格隔开）是不能正确工作的，因为解释器会调用程序`foo` 并将 `=` 和 `bar`作为参数。 总的来说，在shell脚本中使用空格会起到分割参数的作用，有时候可能会造成混淆，请务必多加检查。

Bash中的字符串通过`'` 和 `"`分隔符来定义，但是它们的含义并不相同。以`'`定义的字符串为原义字符串，其中的变量不会被转义，而 `"`定义的字符串会将变量值进行替换。

```shell
foo=bar
echo "$foo"
# 打印 bar
echo '$foo'
# 打印 $foo
```

这里 `$1` 是脚本的第一个参数。与其他脚本语言不同的是，bash使用了很多特殊的变量来表示参数、错误代码和相关变量。下面是列举来其中一些变量，更完整的列表可以参考 [这里](https://www.tldp.org/LDP/abs/html/special-chars.html)。

- `$0` - 脚本名
- `$1` 到 `$9` - 脚本的参数。 `$1` 是第一个参数，依此类推。
- `$@` - 所有参数
- `$#` - 参数个数
- `$?` - 前一个命令的返回值
- `$$` - 当前脚本的进程识别码
- `!!` - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。
- `$_` - 上一条命令的最后一个参数。如果你正在使用的是交互式 shell，你可以通过按下 `Esc` 之后键入 . 来获取这个值。

命令通常使用 `STDOUT`来返回输出值，使用`STDERR` 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 返回码或退出状态是脚本/命令之间交流执行状态的方式。返回值0表示正常执行，其他所有非0的返回值都表示有错误发生。

退出码可以搭配 `&&`（与操作符）和 `||`（或操作符）使用，用来进行条件判断，决定是否执行其他程序。它们都属于短路[运算符](https://en.wikipedia.org/wiki/Short-circuit_evaluation)（short-circuiting） 同一行的多个命令可以用` ; `分隔。程序 `true` 的返回码永远是`0`，`false` 的返回码永远是`1`。让我们看几个例子：

```shell
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

false ; echo "This will always run"
# This will always run
```


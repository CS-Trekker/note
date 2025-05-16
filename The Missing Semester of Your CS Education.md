# 1.课程概览与shell
  
---

## 回到上一个目录
```bash
cd -
```

## ls的一些参数

| 参数           | 作用简述                                  | 示例              |
| ------------ | ------------------------------------- | --------------- |
| `-l`         | 显示详细信息（开头的第一个字符是-,d还是l决定了其类型）         | `ls -l`         |
| `-a`         | 显示**所有文件**，包括隐藏文件（以`.`开头）（-A会去除.和..）  | `ls -a`         |
| `-h`         | 和 `-l` 配合使用，显示更**人类可读**的文件大小（如 KB、MB） | `ls -lh`        |
| `-R`         | 递归显示**所有子目录**的内容                      | `ls -R`         |
| `-r`         | 反转排序（可与 `-t` 等搭配）                     | `ls -ltr`       |
| `-t`         | 按**修改时间排序**（最新的文件排最前）                 | `ls -lt`        |
| `-S`         | 按文件**大小排序**（从大到小）                     | `ls -lS`        |
| `-F`         | 列出文件类型                                |                 |
| `-d */`      | 只显示当前目录下的**文件夹**                      | `ls -d */`      |
| --color=auto | 显示有颜色的输出，便于区分文件类型                     | ls --color=auto |

## shell脚本中的一堆$参数和判断符号

```bash
#!/bin/bash 

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

| 变量       | 含义                               | 示例值                         |
|------------|------------------------------------|--------------------------------|
| `$0`       | 当前脚本的文件名或命令名                | `./myscript.sh`               |
| `$#`       | 传递给脚本或函数的参数数量               | `2`                            |
| `$$`       | 当前 Shell 或脚本的进程号（PID）         | `4567`                         |
| `$!`       | 最近一个后台执行命令的进程号（PID）       | `12345`（如 `sleep 60 &` 后）  |
| `$@`       | 传给脚本的所有参数（每个参数独立保留）     | `"file1.txt" "file2.txt"`     |
| `$*`       | 传给脚本的所有参数（作为单一整体）        | `"file1.txt file2.txt"`       |
| `$?`       | 最近一条命令的退出状态（0 表示成功）       | `0` 或 `1`                    |
| `$_`       | 上一条命令的最后一个参数                   | `file2.txt`（如上条命令参数）  |
| `!!`       | 上一条完整命令（历史命令）                 | `ls -l /home`                 |

| 操作符 | 含义     | 示例                           |
| ------ | -------- | ------------------------------ |
| `-eq`  | 等于     | `[[ 5 -eq 5 ]]` → 为真（true） |
| `-ne`  | 不等于   | `[[ 5 -ne 3 ]]` → 为真         |
| `-gt`  | 大于     | `[[ 5 -gt 3 ]]` → 为真         |
| `-lt`  | 小于     | `[[ 3 -lt 5 ]]` → 为真         |
| `-ge`  | 大于等于 | `[[ 5 -ge 5 ]]` → 为真         |
| `-lt`  | 小于     | `[[ 3 -lt 4 ]]` → 为真         |
| `-le`  | 小于等于 | `[[ 3 -le 3 ]]` → 为真         |

## shebang

shell脚本的shebang写成**#!/usr/bin/bash**或者**#!/bin/bash**，python写成**#!/usr/bin/env python3**,  `!`和第一个`/`不能缺！

## 一个关于sudo权限的命令

```bash
echo 3 | sudo tee brightness
```

不能使用：

```bash
sudo echo 3 > brightness
```

因为，tee是个**程序**，而`>` 不是程序，而是 **shell 内建的重定向操作符**，没有权限在sys里写入、修改brightness文件

你请了管理员来“说出数字 3”（`sudo echo 3`）；然后你自己却尝试“写入受限的系统文件”，权限不足，自然失败。

## 有关rm、rmdir

rmdir只能删除空文件夹，要使用rm -ri myfolder递归交互式删除

`rm -rf` 非常强大也非常危险！（没有提示）

如果你一不小心写错路径，比如：

```bash
rm -rf /
```

那么你的整个系统就可能被删没了

## 重命名

```bash
mv oldname.txt newname.txt
# mv是移动
```

# 2.shell工具与脚本

---
*TLDR上有一些比较有用的关于shell命令的一些案例说明

## 定义了两个函数的shell脚本实例

```bash
#!/bin/bash

marco() {
	echo $(pwd) > ~/study/last_location.txt
	echo "location has been saved $(pwd)"
}
polo() {
	cd $(cat ~/study/last_location.txt)
}
```

使用方法：
1. source marco.sh或者.　marco.sh

   *函数仅在定义时被加载，脚本会在每次被执行时加载。每次修改函数定义，都要重新加载一次*

2. marco

3. polo(到另外一个路径下)

## debug.sh

```bash
#!/bin/bash

touch stdout.log stderr.log
> stdout.log
> stderr.log
count=1
while true ; do
	n=$(( RANDOM % 100 ))

 	if [[ n -eq 42 ]]; then
    		echo "Something went wrong"
    		>&2 echo "The error was using magic numbers" >> stderr.log
    		echo "The program has run $count times." >> stderr.log
		break
 	fi
	count=$((count+1))
	echo "Everything went according to plan" >> stdout.log
done
echo 'stdout:'
cat stdout.log
echo 'stderr:'
cat stderr.log

# 还可以用until实现
```

## 链接

```bash
ln file.txt link.txt
```

这会创建一个名为 link.txt 的硬链接，指向 file.txt 文件

*只有当所有指向文件数据块的硬链接都被删除时，文件的实际数据才会被清除*

```bash
ln -s /home/user/file.txt symlink
```

这会创建一个名为 symlink 的符号链接，指向 /home/user/file.txt

```bash
ln -sf /home/user/file.txt symlink
```

如果已经存在一个文件叫symlink,那么就强制删除symlink，用符号链接覆盖

## 短路运算符

```bash
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

## 命令替换

```bash
diff <(ls foo) <(ls bar)
```

比较文件夹 `foo` 和 `bar` 中文件的区别

## shell的通配

### 通配符

| 通配符     | 含义                  |
| ------- | ------------------- |
| `*`     | 匹配**任意个（包括0个）**字符   |
| `?`     | 匹配**任意单个字符**        |
| `[...]` | 匹配方括号中**列出的任意一个字符** |

对于文件 `foo`, `foo1`, `foo2`, `foo10` 和 `bar`, `rm foo?` 这条命令会删除 `foo1` 和 `foo2` ，而 `rm foo*` 则会删除除了 `bar` 之外的所有文件

### 花括号自动展开

```bash
touch {foo,bar}/{a..h}
```

该命令会创建 foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h 这些文件

## shellcheck

```bash
shellcheck test.sh
```

检查脚本是否有问题

## exec,stat,xargs,sort,tail,echo

```bash
#!/bin/bash

# 输入目录
directory=$1

# 如果未输入目录，使用当前目录
if [[ -z "$directory" ]]; then        # -z表示“是否为空字符”；-n与之相反
  directory="."
fi

# 使用find命令递归查找所有文件，stat获取最后访问时间，按时间排序
find "$directory" -type f -exec stat --format='%X %n' {} \; | sort -n | tail -n 1
```

1. *命令行可以从==参数==或==标准输入==接受输入*

   

   stat 命令需要将每个文件名作为参数传入，不能从标准输入中读取路径，必须使用 find -exec 或 xargs 等方式显式传递文件名，所以以下是错的：

   ```bash
   find . -type f | stat --format='%X %n' {} \; | sort -n | tail -n 1
   ```

   

2. *如果有文件名称里带空格，则使用：*

```bash
find "$directory" -type f -print0 | xargs -0 stat --format "%X %n" | sort -n | tail -n 1
```



`find ... -print0`：`-print0` 会在文件名之间插入一个空字符（null byte `\0`），而不是空格.

`-0` 选项告诉 `xargs` 以空字符为分隔符来处理输入，从而正确处理带有空格、特殊字符等的文件名。



3. **xargs**把前一个命令的输出，变成后一个命令的参数,例如：

```bash
ls *.log | xargs rm
```


4. stat

```bash
stat --format='%X %n' example.txt
stat -c '%X %n' example.txt
# 1623894245 example.txt

# %X：返回文件的最后访问时间（UNIX 时间戳）。
# %n：返回文件的路径。
# %......
```


5. sort的参数：

`-n`：按数值升序排序。
`-r`：按降序排序。
`-d`：字典序排序。
`-u`：去重。

6. echo的参数：

| 选项   | 含义说明                  |
| ---- | --------------------- |
| `-e` | 启用转义字符（解释`\n`、`\t` 等） |
| `-E` | 禁用转义字符（默认行为，等于“原样输出”） |
| `-n` | 不输出末尾的换行符             |

## fd/find查找文件、文件夹

```bash
# 查找所有名称为src的文件夹
fd -t d '^src$'  # 或者fd -t d 'src'
find . -name src -type d
# 查找所有文件夹路径中包含test的python文件
fd -p '.*/test/.*\.py$' -t f
find . -path '*/test/*.py' -type f
# 查找前一天修改的所有文件
fd -t f --changed-within 1d
find . -mtime -1
# 查找所有大小在500k至10M的tar.gz文件
fd -e tar.gz -S +500k -S -10M
find . -size +500k -size -10M -name '*.tar.gz'

# 删除全部扩展名为.tmp 的文件
fd -e tmp -x rm
find . -name '*.tmp' -exec rm {} \;
# 查找全部的 PNG 文件并将其转换为 JPG
fd -e png -x convert {} {.}.jpg # {}表示找到的文件路径,{.}表示去掉扩展名的文件路径
find . -name '*.png' -exec convert {} {}.jpg \;
# 查找最近一次修改的文件
fd -t f -0 | xargs -0 ls -lt | head -1
find . -type f -print0 | xargs -0 ls -lt | head -1 # 当文件数量较多时，find会得出错误结果，解决办法是增加 -mmin 条件
# 搜索当前目录及其所有子目录中的所有HTML文件，然后将它们全部打包并压缩到一个名为html.zip的归档文件中
fd -e html -t f -0 | xargs -0 tar -cvzf html.zip
fd -e html -0 | xargs -0 zip html.zip

find . -type f -name "*.html" -print0 | xargs -0 tar -cvzf html.zip
find . -type f -name "*.html" -print0 | xargs -0 zip html.zip

***
# tar命令创建的是 .tar.gz 格式
# zip命令创建的是 .zip 格式
# tar用于创建、查看和提取归档文件
```

## 查找shell命令

```bash
history | grep find
```

Ctrl+r （配合了fzf，可以搜索历史命令）

在命令前加一个空格，这条命令将不会存进history

```bash
# 假设你曾多次进入 `/home/user/projects/my-awesome-app`，之后你就可以：
z awesome
# 它就会自动跳转到最匹配的 `/home/user/projects/my-awesome-app` 目录
```

## rg/grep查找代码

```bash
# 查找所有使用了 requests 库的文件
rg -t py 'import requests' .
grep -r --include="*.py" 'import requests' .

# 查找所有没有写 shebang 的文件（包含隐藏文件）
rg -u --files-without-match "^#!" .
grep -rL '^#!' .

# 查找所有的foo字符串，并打印其之后的5行
rg foo -A 5 .
grep -r -A 5 'foo' .

# 打印匹配的统计信息（匹配的行和文件的数量）
rg --stats 'abc' .
grep -r -c 'abc' .
# 结果有所不同，rg更详细
```

## ranger文件管理器
### **导航操作** 
| 操作            | 快捷键           | 说明               |
| ------------- | ------------- | ---------------- |
| 向左移动（返回上一级目录） | `h`           | 返回到上一级目录         |
| 向下移动（选择下一个文件） | `j`           | 向下移动选择文件         |
| 向上移动（选择上一个文件） | `k`           | 向上移动选择文件         |
| 进入选中的目录或文件    | `l`           | 进入选中的目录或打开文件     |
| 查看文件详细信息      | `i`           | 查看文件的元数据         |
| 跳转到目录顶部       | `gg`          | 跳转到当前目录的顶部       |
| 跳转到目录底部       | `G`           | 跳转到当前目录的底部       |
### **文件操作** 
| 操作            | 快捷键           | 说明               |
| ------------- | ------------- | ---------------- |
| 删除文件          | `d`           | 删除选中的文件          |
| 复制文件          | `yy`          | 复制选中的文件          |
| 粘贴文件          | `pp`          | 粘贴剪切的文件          |
| 重命名文件         | `r`           | 重命名选中的文件         |
| 移动文件          | `m`           | 移动选中的文件          |
| 剪切文件          | `x`           | 剪切选中的文件          |
| 撤销操作          | `u`           | 撤销上一个文件操作        |
### **搜索和排序**
| 操作            | 快捷键           | 说明               |
| ------------- | ------------- | ---------------- |
| 搜索文件          | `/`           | 搜索文件或目录          |
| 搜索选中文件的所有匹配项  | `*`           | 在当前目录下搜索选中文件的匹配项 |
| 输入命令模式        | `:`           | 输入命令行模式          |
| 按文件名搜索        | `Ctrl+f`      | 按文件名进行搜索         |
| 按文件大小排序       | `Ctrl+s`      | 按文件大小排序          |
### **文件查看与预览**
| 操作            | 快捷键           | 说明               |
| ------------- | ------------- | ---------------- |
| 进入文件或目录       | `Enter`       | 进入选中的目录或打开文件     |
| 文件预览          | `v`           | 在右侧窗格预览选中文件      |
| 退出预览模式        | `Ctrl+q`      | 退出文件预览模式         |
| 在外部程序中打开文件    | `Shift+Enter` | 使用外部程序打开文件（如编辑器） |

# 3.vim 
  
---

## 写入模式相关命令

| 命令 | 说明 |
|------|------|
| a    | 在光标后写入（append） |
| A    | 在当前行末尾写入 |
| h    | 在光标前写入（此为自定义映射，应注意原意是向左移动） |
| H    | 在当前行行首写入（并自动添加空格） |
| o    | 在当前行下方新建空白行并进入写入模式 |
| O    | 在当前行上方新建空白行并进入写入模式 |

## 文件与窗口操作

| 命令            | 说明                |
| ------------- | ----------------- |
| :e + {路径/文件名} | 在新的缓冲区打开新文件       |
| :ls           | 列出所有缓冲区           |
| :b 1          | 切换到缓冲区编号为 `1` 的文件 |
| :bd           | 默认会删除当前缓冲区        |
| :bd 1         | 删除编号为 `1` 的缓冲区    |
| :sp           | 打开上下分屏窗口（同一缓冲区）   |
| :vsp          | 打开左右分屏窗口          |
| Ctrl+w+k      | 切换到上方窗口           |
| Ctrl+w+j      | 切换到下方窗口           |
| Ctrl+g        | 查看状态行（行号、总行数等）    |
| :x            | 保存并退出             |

## 移动光标（行、屏幕、单词级）

| 命令       | 说明                         |
| -------- | -------------------------- |
| 0        | 移动到行首                      |
| $        | 移动到行尾                      |
| ^        | 移动到第一个非空字符                 |
| gg       | 移动到第一行                     |
| G        | 移动到最后一行                    |
| :\<num\> | 移动到第\<num\>行               |
| J        | 移动到屏幕顶部第一行                 |
| L        | 移动到屏幕底部最后一行                |
| M        | 移动到屏幕中间一行                  |
| b        | 移动到上一个单词开头                 |
| w        | 移动到下一个单词开头                 |
| e        | 移动到下一个单词结尾（和E对“单词”的定义有区别）  |
| Ctrl+u   | 向上滚动10行（注意Caps Lock替代Ctrl） |
| Ctrl+d   | 向下滚动10行                    |
| zz/z.    | 将当前行滚动到屏幕的中间               |
| zt       | 将当前光标所在的行滚动到屏幕的顶部          |
| zb       | 将当前光标所在的行滚动到屏幕的底部          |

## 查找与跳转

| 命令           | 说明                 |
| ------------ | ------------------ |
| /xxx         | 向后查找 xxx           |
| ?xxx         | 向前查找 xxx           |
| n            | 跳转到下一个匹配项          |
| N            | 跳转到上一个匹配项          |
| :set ic      | 忽略大小写查找            |
| :set noic    | 区分大小写查找            |
| :set is？     | 查看是否启用“边输入边查找”     |
| :set hls     | 高亮搜索匹配项            |
| /\csomething | 忽略大小写查找 something  |
| fo           | 光标跳到后面第一个“o”       |
| Fo           | 光标跳到前面第一个“o”       |
| to           | 光标跳到后面第一个“o”的前一个字符 |
| To           | 光标跳到前面第一个“o”的后一个字符 |
| %            | 匹配括号跳转             |
| Ctrl+o       | 跳转到上一个光标位置         |
| Ctrl+h       | 跳转到较新光标位置          |

## 修改与删除

| 命令  | 说明                    |
| --- | --------------------- |
| u   | 撤销上一步操作               |
| x   | 删除光标前的字符              |
| X   | 删除光标所在字符              |
| d   | 删除（需结合移动键）            |
| dd  | 删除当前行                 |
| 2dd | 删除两行                  |
| 8dw | 删除光标后8个单词             |
| ch( | 删除括号内内容并进入写入模式（如 ch[） |
| ca( | 删除括号和括号内内容并进入写入模式     |
| c   | 删除并进入写入模式（需配合移动键）     |
| cc  | 删除整行并进入写入模式           |

## 替换与复制粘贴

| 命令 | 说明 |
|------|------|
| r+字符 | 替换光标所在字符，不进入写入模式 |
| R      | 替换多个字符（覆盖） |
| yy     | 复制整行 |
| y + 移动键 | 复制选定区域 |
| p      | 粘贴 |
| "ayy   | 复制当前行到 a 寄存器 |
| "ap    | 从 a 寄存器粘贴 |
| "Ayy   | 追加复制当前行到 a 寄存器 |
| "bd    | 删除当前行并保存到 b 寄存器 |
| :reg   | 查看所有寄存器内容 |

## 替换（substitute）

| 命令              | 说明                                                |
| --------------- | ------------------------------------------------- |
| :s/thee/the/g   | 替换当前行所有的 thee 为 the                               |
| :%s/thee/the/g  | 替换全文中所有的 thee 为 the                               |
| :%s/thee/the/gc | 替换全文 thee，为每次替换提示确认                               |
| :%s/thee/the/   | 替换全文中每行第一个 thee 为 the（当vimrc中set gdefault后默认是g模式） |

## 可视模式与块操作

| 命令      | 说明                  |
| ------- | ------------------- |
| v       | 进入可视模式              |
| V       | 进入可视行模式             |
| Ctrl+v  | 进入可视块模式             |
| :w TEST | 可视模式保存选中文本为 TEST 文件 |
| ~       | 反转选中字符大小写           |

## 外部命令与文件读写

| 命令         | 说明                  |
| ---------- | ------------------- |
| :!ls       | 执行 shell 命令查看文件列表   |
| :!del TEST | 删除 TEST 文件          |
| :r TEST    | 读取 TEST 文件内容插入当前缓冲区 |
| :r !ls     | 把 `ls` 命令输出插入到当前缓冲区 |

## man 手册调用

| 命令  | 说明             |
| --- | -------------- |
| I   | 打开当前单词的 man 手册 |
## less翻页阅读器

| 快捷键         | 功能说明                    |
| ----------- | ----------------------- |
| `Space`（空格） | 向下翻一页                   |
| `b`         | 向上翻一页（back）             |
| `Enter`     | 向下移动一行                  |
| `k` / `j`   | 分别向上 / 向下移动一行（类似 `vim`） |
| `g`         | 跳到文件开头（go）              |
| `G`         | 跳到文件结尾（Great/End）       |
| `/关键词`      | 向下搜索关键词（支持正则）           |
| `?关键词`      | 向上搜索关键词                 |
## 插件
### vim-plug
在你的 `~/.vimrc` 文件中添加你想装的插件，比如：
```vim
call plug#begin('~/.vim/plugged')

Plug 'ctrlpvim/ctrlp.vim'  " CtrlP 插件

call plug#end()
```
然后安装：
```vim
:PlugInstall
```
查看已安装插件：
```vim
:PlugStatus
```
### CtrlP
在vim中Ctrl+p，就可以模糊搜索文件名或路径，enter打开
### Ack.vim
在vim中`:Ack {要查找的代码}`，下方会自动开一个窗口，（默认情况下在当前目录的文件里）查找该代码，例如：
```vim
:Ack Microsoft ~/study
```
### NERDTree
```vim
:NERDTreeToggle
```

 因为键位映射问题，暂时不好用 
### vim-easymotion
| 动作       | 作用                               |
| -------- | -------------------------------- |
| 按下 `s` 键 | 调用 `easymotion-overwin-f`，进入跳转模式 |
| `,m`     | 已被映射成:noh\<CR\>                  |
| 输入目标字符   | 输入一个字符（如 `e`），作为跳转目标             |
| 查看跳转标签   | 所有匹配字符被高亮，并显示跳转标签（如 `fa`）        |
| 输入跳转标签   | 光标瞬间跳转到目标字符的位置                   |

## vi模式下的快捷键

| 快捷键        | 功能说明        |
| ---------- | ----------- |
| `Ctrl + a` | 移动到行首       |
| `Ctrl + e` | 移动到行尾       |
| `Ctrl + u` | 删除从光标到行首的内容 |
| `Ctrl + k` | 删除从光标到行尾的内容 |
| `Ctrl + p` | 上一条历史命令     |
| `Ctrl + n` | 下一条历史命令     |
| `Ctrl + r` | 反向搜索历史命令    |


| 模式切换命令         | 功能说明         |
| -------------- | ------------ |
| `set -o emacs` | 切换为 Emacs 模式 |
| `set -o vi`    | 切换为 vi 模式    |

| 命令         | 功能说明               |
| ---------- | ------------------ |
| `mkdir -p` | 强制新建多级目录，若目录已存在不报错 |

# 4.数据整理

---
## Regex
```regex
[^abc] # 表示不匹配abc
\w     # 等价于 [A-Za-z0-9_]
\W     # \w的补集
\s     # 匹配空白字符，如空格、制表符、换行符、回车符等
\S     # 匹配非空白字符
\d     # 匹配数字字符
\D     # 匹配非数字字符
\b     # 匹配单词边界
```


---

```text
Jan 13 15:00:14 thesquareplanet.com sshd[25062]: Disconnected from invalid user postgres 190.187.67.67 port 50091 [preauth]
Jan 13 21:58:01 thesquareplanet.com sshd[25433]: Disconnected from authenticating user root 65.60.109.219 port 41337
May 07 14:23:45 thesquareplanet.com sshd[18742]: Disconnected from user admin 192.168.1.100 port 51234
```

```bash
cat ssh.log | sed -E 's/^.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/' | sort | uniq -c | sort -nk1,1 | tail -n5 | awk '{print $2}' | paste -sd,
```
## sed
sed是一个流编辑器
`-E` 是开启扩展正则表达式模式（但仍然不支持?表示非贪婪模式）

| 命令            | 作用                                                    |
| ------------- | ----------------------------------------------------- |
| `sort`        | `uniq` 确实是用来去重的，但它只会统计==相邻==的相同项的出现次数                 |
| `uniq -c`     | 统计每个用户名的出现次数，并显示每个用户名出现的次数。`-c` 参数表示输出时在每一行前面加上出现的次数。 |
| `sort -nk1,1` | 对统计出的结果按出现次数进行排序。`-n` 表示按数值排序                         |
| `tail -n5`    | 显示排序后的结果中的最后5行，即显示出现次数最多的5个用户名。                       |

```bash
sort -n -k1,1 -k2,2 -k3,3 filename
# 先以第一列、再以第二列、再以第三列，进行排序
```

此处的输出结果是：
```text
1951 user
3326 admin
3345 test
3782 123456
10892 root
```
## awk
awk是一个默认基于列的文本处理工具

| 命令                 | 作用                                                              |
| ------------------ | --------------------------------------------------------------- |
| `awk '{print $2}'` | 使用 `awk` 提取每一行的第二列（即用户名）。                                       |
| `paste -sd,`       | 将所有提取出的用户名合并为一行，每个用户名之间用`,`分隔。`-s` 表示将所有输入合并为一行，`-d,` 表示分隔符为`,` |
paste的输入：
```text
user
admin
test
123456
root
```
paste的输出：
```text
user,admin,test,123456,root
```

## awk编程
 ```bash
cat ssh.log | sed -E 's/^.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/' | sort | uniq -c | awk '$1 == 1 && $2 ~ /^c.*e$/ { print $0 }' | wc -l
# 从ssh.log中筛选出：所有仅出现一次，且用户名以 c 开头、以 e 结尾的 SSH 断开连接记录，并打印该用户名及其出现次数。
# 加上 wc -l后就是打印最终结果的行数
```
 标准的 `awk` 用法格式:    `awk '条件 {操作}'`
 
 awk中，`变量 ~ /正则表达式/`表示“这个变量的值是否匹配该正则表达式”，`!~`表示不匹配， 例如：
```bash
awk '$1 ~ /\.sh$/ { print }'

cat last3.log | sed -E 's/^.*: (.*)$/\1/' | sort | uniq -c | sort | awk '$1 != 3 { print > "difference.txt"; print }'
# {print}默认是{print $0}的意思
```

一个更为复杂的awk编程：
```bash
cat ssh.log | sed -E 's/^.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/' | sort | uniq -c | awk 'BEGIN {rows = 0} $1 == 1 && $2 ~ /^c.*e$/ {rows += 1} END {print rows}'
```
## bc（伯克利计算器）
```bash
cat ssh.log | sed -E 's/^.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/' | sort | uniq -c | awk '$1 != 1 {print $1}' | paste -sd+ | bc -l
# 统计非单次登录用户的登录总次数
```

## R语言（统计）
*(需要下载r-base软件包)*
```bash
cat ssh.log | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( \[preauth\])?$/\2/' | sort | uniq -c | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```
输出：
 Min.   1st Qu.   Median     Mean   3rd Qu.     Max. 
  1.000   1.000    1.000     1.828    1.000      8.000 

## gnuplot（可视化）
```bash
cat ssh.log | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' | sort | uniq -c | sort -nk1,1 | tail -n10 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## 清理空间的命令
```bash
sudo apt autoremove    # 删除不需要的依赖
sudo apt clean         # 清理软件包缓存
sudo apt-get autoremove --purge   # 删除不需要的依赖和配置文件

sudo apt update && sudo apt upgrade
```

## tr、sed的y命令、POSIX符号、grep的-v、regex双重筛选
```bash
cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "'s$" | wc -l

cat /usr/share/dict/words | sed 'y/ABCDEFGHIJKLMNOPQRSTUVWXYZ/abcdefghijklmnopqrstuvwxyz/' | grep -E "^([^a]*a){3}.*$" | grep -v "'s$" | wc -l

# 统计words文件中包含至少三个a且不以's结尾的单词个数
```
因为grep不支持lookback，否则用下式就可以匹配了：
```regex
^([^a]*a){3}.*(?<!'s)$
# (?<!PATTERN)表示：当前位置不能匹配PATTERN
```
## 原地备份
```bash
sed -i.bak 's/regex/SUBSTITUTION/' input.txt
# 可以自动创建一个后缀为 .bak 的备份文件
```
## journalctl日志
```bash
journalctl -q --list-boots # -q表示不要弹出'Hint:...'
#列出所有的启动记录

journalctl -q -b
# 默认是查看当前启动的日志，不需要加 0

journalctl -q -b -1 
# 用来查看上一次启动的日志

journalctl --since "2015-01-10" --until "2015-01-11 03:00"
# 省略了时间，默认是00:00:00，省略了秒数
journalctl --since 09:00 --unitl "1 hour ago"

sudo systemd-analyze plot > systemd.svg
#　使用 systemd-analyze 工具看一下启动时间都花在哪里
eog systemd.svg # eog是默认的图形界面的图片查看器，此外feh是命令行工具，更适合放进脚本中

```

```bash
# startuptime.txt可以通过写一个getlog.sh获得
$ cat startuptime.txt
5月 08 09:13:22 ustclug-linux101 systemd[943]: Startup finished in 83ms.
5月 08 16:08:31 ustclug-linux101 systemd[942]: Startup finished in 102ms.
5月 08 00:15:35 ustclug-linux101 systemd[1044]: Startup finished in 103ms.
5月 07 20:57:28 ustclug-linux101 systemd[944]: Startup finished in 108ms.
5月 07 16:35:02 ustclug-linux101 systemd[943]: Startup finished in 107ms.
5月 06 23:09:10 ustclug-linux101 systemd[946]: Startup finished in 94ms.
5月 06 18:41:56 ustclug-linux101 systemd[947]: Startup finished in 105ms.
5月 06 00:27:53 ustclug-linux101 systemd[950]: Startup finished in 98ms.
5月 06 00:26:06 ustclug-linux101 systemd[945]: Startup finished in 101ms.
5月 06 00:24:54 ustclug-linux101 systemd[943]: Startup finished in 101ms.

$ sed -i.bak -E 's/^.*in ([[:digit:]]*)ms.$/\1/' startuptime.txt
83
94
98
101
101
102
103
105
107
108

# 平均数：
cat  startuptime.txt | paste -sd+  | bc | awk '{print $1/10}'
#　中位数：
cat startuptime.txt | paste -sd\  | awk '{print ($5+$6)/2}'
```
## sed的d用法
```text
test.txt:
Line 1
Start
Line 2
Line 3
End
Line 4
```
```bash
sed '/Start/,/End/d' test.txt
Line 1
Line 4
```
```bash
sed '0,/End/d' test.txt 
Line 4

sed '/Start/,$d' test.txt 
Line 1
```
*这些sed只有使用-i后才能修改原文件（但是就不会有stdout了）*

# 5.命令行环境
## job control（任务控制）
```python
#!/usr/bin/env python3
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1

# 这个python脚本的写法暂时没掌握
```
### 信号

| 信号名称      | 信号编号 | 描述                       | 特殊快捷键 |
| --------- | ---- | ------------------------ | ----- |
| `SIGHUP`  | 1    | 挂起信号（通常由`终端断开`触发）        | 无     |
| `SIGINT`  | 2    | 中断信号                     | `^C`  |
| `SIGQUIT` | 3    | 退出信号（会生成核心转储文件core dump） | `^\`  |
| `SIGKILL` | 9    | 强制终止信号（`无法捕获或忽略`）        | 无     |
| `SIGTERM` | 15   | 终止信号（请求进程`优雅`终止）         | 无     |
| `SIGCONT` | 18   | 恢复信号(fg、bg)              | 无     |
| `SIGSTOP` | 19   | 停止信号（`暂停`进程执行，`无法捕获或忽略`） | 无     |
| `SIGTSTP` | 20   | 停止信号（`暂停`进程执行）           | `^Z`  |
*SIGHUP不会直接导致进程终止，但进程接收到SIGHUP后一般会自行终止*
```bash
nohup sleep 3000 &
jobs -l                                 # 查看当前终端的进程
kill -l                                    # 查看所有信号的编号
kill <PID>                            # kill默认是发送SIGTERM信号
kill -9 <PID>                       # 默认是SIGKILL信号
kill -CONT <PID>                # 发送SIGCONT信号
fg %1                                    # 把进程1恢复到前台
bg %1                                   # 把进程1恢复到后台
ps aux #a，x组合起来——查看所有进程，u——以用户友好的形式显示
```
*一个作业（job)可以是一个单独的进程，也可以是一组通过 管道 | 连接起来的多个进程*
### pgrep
```bash
pgrep <命令名>(例如sleep)
# 3959

pgrep -f "sleep 10000"
pgrep -f 10000       # 10000在这里叫做'命令行参数'
# 3959

pgrep -a sleep
# 3959 sleep 10000

pgrep -af sleep
pgrep -af 10000
# 3959 sleep 10000
```
### pkill
```bash
# 假如我有3个后台进程，sleep 100 &,sleep 200 &，sleep 1000 &
pkill -f 100
# 关闭sleep 100 &和sleep 1000 &
```
### 进程同步
```bash
#！/usr/bin/bash
pidwait()
{
   while kill -0 $1 do
   sleep 1 
   done
   ls
}
```
*while 判断的是命令行的返回值而不是布尔值，这个和其他语言有所区别*
*当进程不存在时，kill -0 返回值是非 0， 表示失败false*
## terminal multiplexers(tmux)（终端多路复用器）
### Window快捷键
| 快捷键       | 功能说明              |
| --------- | ----------------- |
| `C-a c`   | 新建一个窗口（window）    |
| `C-a ,`   | 重命名当前窗口           |
| `C-a &`   | 关闭当前窗口（会提示确认）     |
| `C-a l`   | 切换到上一次活动的窗口       |
| `C-a p`   | 切换到前一个窗口（3→2→1）   |
| `C-a n`   | 切换到下一个窗口          |
| `C-a 0~9` | 快速切换到指定编号的窗口（0-9） |
| `C-a w`   | 显示所有窗口列表并选择       |

### Pane快捷键
| 快捷键               | 功能说明                         |
| ----------------- | ---------------------------- |
| C-a \|            | 将当前窗格纵向分割为左右两个 pane          |
| `C-a -`           | 将当前窗格横向分割为上下两个 pane          |
| `C-a x`           | 关闭当前窗格（kill pane，会提示确认）      |
| `C-a o`           | 切换到下一个 pane                  |
| `C-a ;`           | 在最近使用过的两个 pane 之间快速切换        |
| `Alt <arrow>`     | 切换pane                       |
| `C-a <arrow>`     | 切换pane                       |
| `C-a z`           | 当前 pane 全屏/恢复原状（Zoom）        |
| `C-a <Space>`     | 在不同的面板排布间切换                  |
| `C-a C-<arrow>`   | 调整当前 pane 的大小(1字符)（现在可以用鼠标了） |
| `C-a Alt-<arrow>` | 调整当前 pane 的大小(5字符)（现在可以用鼠标了） |
| `C-a q`           | 显示所有 pane 的编号，按数字键可快速跳转      |
| `C-a !`           | 将当前 pane 弹出为一个新窗口            |

### Session和其他快捷键
| 快捷键              | 功能说明           |
| ---------------- | -------------- |
| `C-a d`          | 分离当前会话(仍在后台运行) |
| `C-a s`或者`C-a w` | 切换会话           |
| `C-a $`          | 重命名session     |
| `C-a r`          | 重新加载 tmux 配置文件 |
| `C-a C-a`        | 光标移动到行首        |

```bash
tmux ls
# 列出当前正在运行的session

tmux attach -t 0
# 连接到名为“0”的session（-t是target)

tmux new -s "session 1" -n "window 1"
#  创建名为“session 1”的新session,第一个window名为“window 1”
tmux neww -n "window 2"
# 创建window 2

tmux kill-session -t 0
# 终结session

tmux rename-session -t 0 "new name"
# 重命名session，也可以进去用快捷键

tmux source-file ~/.tmux.conf
# 重新加载 tmux 配置文件
```

## dotfiles
*很多程序例如Anaconda、Go、Rust，都要求您在 shell 的配置文件中包含一行类似`export PATH="$PATH:/path/to/program/bin`的命令*
*bash本身不会控制字体大小，终端模拟器来控制，我使用的是Xfce Terminal*
### alias
| 方式           | 作用                | 是否永久   |
| ------------ | ----------------- | ------ |
| `\ls`        | 临时忽略 alias，执行原始命令 | 一次性    |
| `unalias la` | 删除一个别名            | 当前会话生效 |
*alias ll='ls -alhF'；alias la='ls -A'；alias lla='la -l'；alias l='ls -CF'；alias v='vim'；alias sl='ls'*
## remote machine（服务器）
# 6.版本控制（Git)
## Git命令集合
### 暂存与diff
```bash
git add -p
# 交互式地添加修改到暂存区（patch 模式）
git restore --staged file
# 撤销 git add，不会修改内容
git restore file
# 撤销你还没 git add 的修改

git diff
# 比较工作区与暂存区的差异（未暂存的改动）
git diff --staged
# 显示的是 暂存区 和 上一次提交（即最新的 commit）之间的差异

git diff <commit-id> HEAD <file>
# 比较指定 commit 与当前 HEAD 之间某个文件的差异

git diff <file>
# 比较工作区与暂存区中某个文件的差异

git diff HEAD <file>
# 比较工作区与最近一次提交（HEAD）之间某个文件的差异

# 举例说明差异：
# 修改 hello.txt 后，还未执行 git add
git diff hello.txt             # 有差异
git diff HEAD hello.txt        # 有差异

# 执行 git add hello.txt 后
git diff hello.txt             # 无差异（已经加入暂存区）
git diff HEAD hello.txt        # 仍有差异（暂存区未提交）
```
### 撤销操作
| 命令                             | 说明                                                                                |
| ------------------------------ | --------------------------------------------------------------------------------- |
| `git restore <file>`           | 可以撤销工作区中某个**文件**的修改                                                               |
| `git checkout -f <commit-id>`  | 强制丢弃当前工作区的所有（未暂存的）修改并切换到指定的 commit，进入 detached HEAD 状态（此时你不是在某个分支上，而是直接指向某个具体的提交） |
| `git reset --hard <commit-id>` | 将当前分支指针（HEAD）移动到指定的提交；<br>同时将暂存区和工作区都重置为该提交的状态；<br>会丢弃所有之后的工作区、暂存区、提交区内容          |
| `git revert <branch_name>`     | 用一个新的“反向提交”来撤销指定提交的内容；<br>不会改变原有历史，适合在多人协作时使用，安全且可恢复                              |
### 分支操作
```bash
git branch
# 查看本地所有分支

git branch -vv
# 查看分支的详细信息，包括 tracking 信息和最后一次提交

git branch <branch-name>
# 创建新分支

# 相对引用；强制修改分支指向的位置
git branch -f main HEAD~3
git branch -f main HEAD^

git branch -d <branch-name>
# 只能删除已经被合并到当前分支的分支（除非-D)

git branch --merged/--no-merged
git branch --merged/--no-merged <branch-name>
查看哪些分支已经（或 还未）合并到当前分支（或 指定分支）
```

```bash
git checkout <branch-name>
# 切换到已有分支

git checkout -b <branch-name>
# 创建并切换到新分支
git checkout -b side origin/main
# 基于 origin/main创建了一个名为 side 的新分支，建立 side和origin/main的跟踪关系

git checkout <commit-id>
# 切换到某次提交（进入 detached HEAD 状态）

git checkout -f <commit-id>
```
### 合并操作
```bash
git merge <branch-name>
# 将指定分支合并到当前分支

git mergetool
# 打开vimdiff，在下方窗口编辑来进行解决冲突，用:wqa退出（三个字母有顺序）

git merge --abort
# 中止合并过程（仅限冲突未解决时）
git merge --continue
# 继续未完成的合并过程
```
### 变基操作
*建议先多用 `merge`,只对==尚未推送或分享给别人==的==本地==修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作*
![[Pasted image 20250513092230.png|500]]
```bash
git rebase --onto master server client
# 把 client 中“从 server 开始的提交”剪下来，接到 master 后面
```
![[Pasted image 20250513092440.png|500]]
```bash
git checkout master
git merge                           # 进行一次fast-forward合并
git rebase master             # 把当前分支变基到master上
git rebase master server # 将主题分支 （即本例中的 server）变基到目标分支（即 master）上
```
![[Pasted image 20250513093345.png]]

	
```bash
git rebase -i main # 交互式变基（可挑选<commit>，可以交换位置）
A←B←C←D
# 如果在D上直接使用git rebase A，是没有结果的；但可以使用git rebase -i A，通过交互来将D加到A上

git pull --rebase   # 相当于git fetch + git merge origin/当前分支
```
### cherry-pick操作
```bash
git cherry-pick <commit1> <commit2> <commit3>……
# 挑选你想要的提交，复制到当前分支上来
```
### 远程仓库操作
```bash
git init --bare
# 创建裸仓库(用于作为远程仓库)

git remote add <name> <url>
# 例如git remote add origin ~/gitlearn/remote
```
``` bash
###### 分支的关联关系 #####

git push <remote> <local branch>:<remote branch>
# 例如 git push origin master:master
# 推送本地的master分支来更新远程仓库上的master分支

# 建立本地分支和远程分支的关联关系,这个关联结果可通过git branch -vv查看
git checkout --track -b <new_branch_name> <origin>/<remote_branch_name>
# 创建一个本地分支来跟踪远程的分支

git branch -u origin/master
# 设置或修改当前所在的本地分支的上游分支

# 如果本地没有名为 fix 的分支，而远程仓库中存在一个名为 fix 的分支，
# 那么执行 git checkout fix 时，Git 会自动创建一个本地 fix 分支，并将其设置为跟踪远程的 origin/fix 分支。

# 可通过简写 @{u} 来引用当前分支（要求其设置好跟踪分支）的上游分支
```

```bash
git remote -v                                    # 显示URL
git ls-remote <远程仓库名>           # 查看远程仓库当前有哪些分支及其 commit
git remote show <远程仓库名>      # 获得远程分支的更多信息
git remote rename <remote> <new_remote>
git remote rm <远程仓库名>           # 移除远程仓库

git clone <url> <folder_name>
# 例如 git clone ~/gitlearn/remote ~/gitlearn/demo2
git pull = git fetch ; git merge

git rm file
# 停止跟踪该文件，同时删除本地文件
git rm --cached file
# 停止跟踪该文件，同时保留本地文件（如果该文件已在暂存区，则也会从暂存区中移除）

git mv file_from file_to
# 移动或重命名文件,相当于：
mv file_from file_to + git rm file_from + git add file_to

git branch -m old_name new_name
# 重命名本地分支，要确保没有在要重命名的分支上

git push origin --delete serverfix
# 删除远程分支serverfix

git push <远程仓库名> <分支名>
# 例如 git push origin main
# 把本地的 main 分支的代码，推送到远程仓库 origin 的 main 分支上

git push <远程仓库名> <本地位置>:<远程分支名>
# 例如 git push origin foo^:new_branch
# 当指定远程分支不存在时会自动新建
```
### 标签操作
```bash
git tag                                                                          # 查看所有标签
git tag -l "v1.8.5*"                                                       # 查找标签(此处使用了通配符*)
git tag <标签名>                                                         # 轻量标签
git tag -a <标签名> -m "标签描述"                           # 附注标签
git tag -a <标签名> <commit_id>                             # 没有-m，会打开vim编辑器
git tag -d <标签名>                                                    # 删除标签
git push --tags                                                            # 将所有标签推送到远程仓库
git show <标签名>                                                      # 查看标签信息

git describe <commit/branch>
```
### log/blame操作
```bash
git log -p # 可以显现每次提交所引入的差异
git log --all --graph --decorate
# 显示所有分支的提交历史，以图结构展示（有向无环图 DAG）
git log --all --graph --decorate --oneline
# 简洁图形格式展示，每条提交信息一行（常设 alias: gl）
git log --pretty=format:"%h - %an, %ar : %s"
# ca82a6d - Scott Chacon, 6 years ago : changed the version number
# 2c4c630 - CS-Trekker, 33 分钟前 : Merge remote-tracking branch 'refs/remotes/origin/master'
git log -1 README.md
# 查看 README.md 文件最近的一次提交记录


git blame <file>
# 查看文件每一行的提交记录及作者
git show <commit-id>
# 查看指定提交的内容和变更
git show --pretty=format:"%s" <commit_id> | head -1
# 查看某次提交（commit）信息的第一行提交说明的标题部分

# 提交信息的写法
# 第一行：简洁明了的提交标题（不超过50个字符）
# 第二行：空一行
# 后续：详细描述本次提交做了什么、为什么这么做
```
### stash操作
```bash
git stash
# 临时保存当前工作区和暂存区的变动（保存到栈中），当前分支回到最近一次提交的状态

git stash push -m "描述"

git stash list
# stash@{0}: On master: edition v2
# stash@{1}: On master: edition

git stash pop
# 恢复最近一次 stash 的内容，并从栈中移除( FILO :First In, Last Out)
git stash pop stash{1}

git stash drop stash{1} # 删除stash{1}
```
### 其他实用操作
```bash
git cat-file -p <commit-id>
# 查看指定提交对象的详细信息

git commit -a
# 自动把所有已经跟踪过的文件暂存起来一并提交
git commit --amend
# 刚提交后发现忘记git add一个文件时

git bisect
# 二分查找哪个提交引入了 bug（适用于回归定位）
```
## SSH 连接 GitHub 测试
```bash
ssh -T git@github.com
# 测试 SSH 连接是否成功

# 若出现 Permission denied (publickey) 错误，则添加对应私钥到 ssh-agent：
ssh-add ~/.ssh/yes
```
## 基于历史版本开发新分支
```bash
# checkout方法
	# 1. 切换到目标 commit（历史版本）
	git checkout 698281bc68
	# 2. 基于这个 commit 创建并切换到一个新分支
	git checkout -b <new-branch-name>

# switch方法
	# 直接一步完成：基于某个 commit 创建并切换到新分支
	git switch -c <new-branch-name> 698281bc68
```
## 错误地提交敏感文件
*（如 `~/gitlearn/demo/privacy.txt`）*
```bash
cd ~/gitlearn/demo

git filter-repo --path privacy.txt --invert-paths
# 运行以下命令，删除 Git 历史中包含 privacy.txt 的提交
# --path privacy.txt  指定你要删除的文件路径
# --invert-paths        反转路径逻辑，表示移除所有包含该路径的提交

git reflog expire --expire=now --all
# 删除 Git 的本地引用日志，防止通过 reflog 恢复
git gc --prune=now
# 清除未被引用的 Git 对象（也就是被“删掉”的文件在历史中的痕迹）

git push origin --force --all
git push origin --force --tags
# 如果你已经将这份历史推送到远程仓库，并且需要更新远程仓库的历史，可以执行以下命令进行强制推送
```
## pull request步骤
1. Fork（复制别人的项目到自己的账户），或者是项目的成员直接使用；使用 git clone 命令把项目拉到本地。
2. 创建并切换到一个新分支用于开发你自己的功能，保持主分支整洁
```bash
git checkout -b <branch-name>
```
3. 修改代码并提交（Commit）
4. 推送到 GitHub
5. 创建 Pull Request（在自己fork来的repository那个界面）
6. 参与讨论 & 等待审核
## fugitive.vim插件
```vim
#可在vim界面直接使用Git命令
:Git commit -m "commit message"

:Gdiffsplit              # 对比工作区和暂存区的文件差异
:Gwrite                   # 相当于git add
:Gread                    # 从 Git 中“恢复”文件，撤销未提交修改
:Gedit HEAD~1:%  # - 打开当前文件的上一个版本（新缓冲区）

:G                            # 进入状态窗口
按 s                         # 暂存文件（相当于 git add）
按 u                         # 取消暂存（相当于 git reset）
按 P                         # 推送（相当于git push)
按 X                         # 丢弃修改（相当于 git checkout -- <file>）
按 cc                       # 提交更改（直接进入提交信息编辑界面）
按 R                         # 刷新状态
按 gq 或 :q             # 退出界面
```

# 7.调试及性能分析
## 调试
### 日志
*日志文件大多位于/var/log/文件夹下*
*lnav    一个交互式、彩色的日志查看器*
```bash
printf "\e[38;2;255;0;0mThis is red\e[0m\n"    

# \e[38;2;255;0;0m和\e[0m是ANSI转义序列
# 38：表示设置 前景色,2：表示使用 RGB 模式
# 255;0;0：对应 RGB 中的红色（255红，0绿，0蓝）
# \e[0m表示重置所有样式
# \n表示换行，printf不是默认换行

echo -e "\e[38;2;255;0;0mThis is red\e[0m"
# 默认换行
```
```bash
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done

printf "\n"

# █+38（前景色）、空格+48（背景色）两种写法都可以
```
```bash
logger "hello"            # 把“hello”加入系统日志
journalctl --since "1m ago" | grep Hello
```
```bash
sudo dmesg -T | grep -i error    # 筛选内核日志中所有包含“error”（忽略大小写）的信息
```
### 调试器debugger
```bash
python3 -m ipdb bubble.py
# 使用ipdb调试
```

| ipdb命令              | 说明                                                    |
| ------------------- | ----------------------------------------------------- |
| `l`                 | 显示当前执行位置附近的源代码（默认显示11行），可以用 `l [start],[end]` 指定显示范围。 |
| `s`                 | 单步执行，进入函数内部。                                          |
| `c`                 | 继续执行程序直到遇到断点或程序结束。                                    |
| `r`                 | 继续执行直到当前函数返回。                                         |
| `n`                 | 单步执行，但不进入函数内部（跳过函数调用）。                                |
| `q`                 | 退出调试器，终止程序执行。                                         |
| `restart`           | 重新启动调试程序，从头开始执行。                                      |
| `b` (break)         | 设置断点，例如 `b 6` 表示在第6行设置断点。                             |
| `cl` (clear)        | 清除断点，可以用 `cl [断点号]` 清除指定断点，或者 `cl` 清除所有断点。            |
| `p`                 | 打印表达式的值，例如 `p variable`。                              |
| `pp` (pretty print) | 美化打印表达式的值，更易阅读。                                       |
| `p local()`         | 打印当前作用域内的局部变量字典。                                      |
### 系统调用strace
```bash
sudo strace ls -l > /dev/null # 跟踪 ls -l 命令的系统调用过程
```
### 静态分析
*英语静态分析工具writegood*
```python
import time

def foo():
    return 42

for foo in range(5):
    print(foo)
bar = 1
bar *= 0.2
time.sleep(60)
```
#### pyflakes
```bash
pyflakes lint.py
# lint.py:6:5: redefinition of unused 'foo' from line 3
# lint.py:11:7: undefined name 'baz'
```
#### mypy
```bash
mypy lint.py
# lint.py:6: error: Incompatible types in assignment (expression has type "int", variable has type "Callable[[], Any]")  [assignment]
# lint.py:9: error: Incompatible types in assignment (expression has type "float", variable has type "int")  [assignment]
# lint.py:11: error: Name "baz" is not defined  [name-defined]
# Found 3 errors in 1 file (checked 1 source file)
```
#### Vim插件ALE
```vim
#######  把分析器集成到编辑器中  #######
:ALEToggle                  # 切换 ALE 的开启/关闭状态 
```
### Python虚拟环境(venv)
```bash
python3 -m venv venv           # 在当前目录下创建一个名为 venv 的文件夹
source venv/bin/activate    # 激活虚拟环境(alias=act)
pip install <module>
deactivate                             # 退出虚拟环境
```
## 性能分析
```python
import time, random
n = random.randint(1, 10) * 100

# 获取当前时间 
start = time.time()

# 执行一些操作
print("Sleeping for {} ms".format(n))
time.sleep(n/1000)

# 比较当前时间和起始时间
print(time.time() - start)

# Output
# Sleeping for 500 ms
# 0.5713930130004883(CPU占用问题)
```

| 名称              | 含义简述                               |
| --------------- | ---------------------------------- |
| **Real Time**   | 程序从开始到结束所经历的**真实墙钟时间**（以上程序所测得的时间） |
| **User Time**   | CPU 执行用户代码所花的时间（不包括系统调用等）          |
| **System Time** | CPU 执行内核代码所花的时间（处理系统调用等）           |
```bash
time curl https://github.com &> /dev/null

# real	0m1.084s（大部分时间用在等待网络响应）
# user	0m0.047s
# sys	0m0.013s
```
### 分析器（CPU分析器）
| 特性           | 跟踪分析器（Tracing Profiler）         | 采样分析器（Sampling Profiler）       |
|----------------|------------------------------------------|----------------------------------------|
| 数据收集方式   | 函数调用插桩，记录每次调用               | 定期采样当前调用栈                     |
| 精度           | 高，能精确记录每次函数调用时间和路径     | 低，统计概率性信息                     |
| 对程序性能影响 | 较大（插桩和记录操作频繁）               | 较小（采样频率可控）                   |
| 适用场景       | 调试、开发阶段精确定位问题               | 生产环境、整体性能趋势分析             |
| 对调用栈支持   | 强，能还原完整调用路径                   | 弱，仅提供采样时的调用栈快照           |
```python
#　一个用来模拟grep命令的python脚本，需要至少三个参数（重复执行次数、查找的pattern、查找的文件），此处用来作为分析的对象
#!/usr/bin/env python

import sys, re

def grep(pattern, file):
    with open(file, 'r') as f:
        print(file)
        for i, line in enumerate(f.readlines()):
            pattern = re.compile(pattern)
            match = pattern.search(line)
            if match is not None:
                print("{}: {}".format(i, line), end="")

if __name__ == '__main__':
    times = int(sys.argv[1])
    pattern = sys.argv[2]
    for i in range(times):
        for file in sys.argv[3:]:
            grep(pattern, file)
```
#### cProfile（函数级别）
```bash
python3 -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py
# cProfile是Python内置的跟踪分析器（缺点：输出太多，可读性差）
```
#### line_profiler（行级别）
```bash
kernprof -l -v urls.py
# urls.py是访问指定网页，解析其 HTML 内容，并提取页面中所有链接的 URL
#　line_profiler是逐行跟踪分析器，要配合在代码中插入装饰器@profile使用，用于定位函数内部“性能瓶颈”

#######################################
Wrote profile results to urls.py.lprof
Timer unit: 1e-06 s

Total time: 0.474917 s
File: urls.py
Function: get_urls at line 7

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     7                                           @profile
     8                                           def get_urls():
     9         1     465727.0 465727.0     98.1      response = requests.get('https://missing.csail.mit.edu')
    10         1       8781.6   8781.6      1.8      s = BeautifulSoup(response.content, 'lxml')
    11         1          0.3      0.3      0.0      urls = []
    12        48        389.0      8.1      0.1      for url in s.find_all('a'):
    13        47         18.7      0.4      0.0          urls.append(url['href'])

```
### 内存分析器
#### memory_profiler（行级别）
```bash
python3 -m memory_profiler mem.py
######################################
Filename: mem.py

Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     1   22.500 MiB   22.500 MiB           1   @profile
     2                                         def my_func():
     3   30.125 MiB    7.625 MiB           1       a = [1] * (10 ** 6)
     4  182.750 MiB  152.625 MiB           1       b = [2] * (2 * 10 ** 7)
     5   30.230 MiB -152.520 MiB           1       del b
     6   30.230 MiB    0.000 MiB           1       return a
```
### perf
```bash
sudo perf stat stress -c 1
# 使用 stress 工具对系统进行 1 个 CPU 核心的压力测试，并通过 perf 的stat工具收集其运行过程中的性能统计数据
```

# 8.元编程
# 9.安全和密码学

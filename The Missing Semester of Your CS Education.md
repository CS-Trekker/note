# 1.课程概览与shell
> `whoami`或者`echo $USER`查看当前用户名
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

| 变量   | 含义                    | 示例值                       |
| ---- | --------------------- | ------------------------- |
| `$0` | 当前脚本的文件名或命令名          | `./myscript.sh`           |
| `$#` | 传递给脚本或函数的参数数量         | `2`                       |
| `$$` | 当前 Shell 或脚本的进程号（PID） | `4567`                    |
| `$!` | 最近一个后台执行命令的进程号（PID）   | `12345`（如 `sleep 60 &` 后） |
| `$@` | 传给脚本的所有参数（每个参数独立保留）   | `"file1.txt" "file2.txt"` |
| `$*` | 传给脚本的所有参数（作为单一整体）     | `"file1.txt file2.txt"`   |
| `$?` | 最近一条命令的退出状态（0 表示成功）   | `0` 或 `1`                 |
| `$_` | 上一条命令的最后一个参数          | `file2.txt`（如上条命令参数）      |
| `!!` | 上一条完整命令（历史命令）         | `ls -l /home`             |

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
>TLDR上有一些比较有用的关于shell命令的一些案例说明
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

## 命令/进程替换、diff和cmp

```bash
diff <(ls foo) <(ls bar)         # "<"后面不能有空格
# 比较目录foo和bar中文件的区别，用两个命令的结果生成了临时文件，然后比较两个文件的不同
```

```bash
cmp <(l Potpourri/) <(l Debugging\ and\ Profiling/)
# /dev/fd/63 /dev/fd/62 不同：第 8 字节（第 1 行）
# 这里用cmp结果不明显
```
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
>fd好像改名fdfind，为避免冲突

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
```

### tar和zip
>tar 是`归档`工具，负责将多个文件打包成一个 .tar 文件，加上 -z 选项后会用 `gzip`(“只能压缩不会归档”) 压缩，生成`.tar.gz`文件
>zip命令可以`归档`和`压缩`,创建的是 `.zip` 格式
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
> vim的宏还没掌握
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

| 命令            | 说明                       |
| ------------- | ------------------------ |
| /xxx          | 向后查找 xxx                 |
| ?xxx          | 向前查找 xxx                 |
| n             | 跳转到下一个匹配项                |
| N             | 跳转到上一个匹配项                |
| :set ic       | 忽略大小写查找                  |
| :set noic     | 区分大小写查找                  |
| :set is？      | 查看是否启用“边输入边查找”           |
| :set hls      | 高亮搜索匹配项                  |
| /\csomething  | 忽略大小写查找 something        |
| fo            | 光标跳到后面第一个“o”             |
| Fo            | 光标跳到前面第一个“o”             |
| to            | 光标跳到后面第一个“o”的前一个字符       |
| To            | 光标跳到前面第一个“o”的后一个字符       |
| %             | 匹配括号跳转                   |
| Ctrl+o        | 跳转到上一个光标位置               |
| Ctrl+h        | 跳转到较新光标位置<br>（有问题、暂时不能用） |
| m+\<letter\>  | 在当前光标位置打上字母标签            |
| \`+\<letter\> | 光标跳转到字母标签的位置             |

## 修改与删除

| 命令          | 说明                    |
| ----------- | --------------------- |
| x           | 删除光标前的字符              |
| X           | 删除光标所在字符              |
| d           | 删除（需结合移动键）            |
| dd          | 删除当前行                 |
| 2dd         | 删除两行                  |
| 8dw         | 删除光标后8个单词             |
| ch(         | 删除括号内内容并进入写入模式（如 ch[） |
| ca(         | 删除括号和括号内内容并进入写入模式     |
| c           | 删除并进入写入模式（需配合移动键）     |
| cc          | 删除整行并进入写入模式           |
| u           | 撤销上一步操作               |
| :earlier 10 | 撤销10次操作               |
| :earlier 5m | 回到5分钟前的状态             |
| :later 2m   | 在已撤销的基础上，向前恢复2分钟的更改   |
> `d/\<PATTERN\>`可以删除从光标处到下一处匹配 pattern 的字符串

> /\<PATTERN\>，使用命令`cgn`会选中当前光标之后的第一个匹配项，删除此匹配项，进入插入模式，让你输入新内容，例如可以将foo替换成fuu，如果再按下 `.` ，可以重复此动作（自动将下一个foo换成fuu)
> 与此类似的有`cgN`、`dgn`、`ygn`

## 替换与复制粘贴

| 命令      | 说明               |
| ------- | ---------------- |
| r+字符    | 替换光标所在字符，不进入写入模式 |
| R       | 替换多个字符（覆盖）       |
| yy      | 复制整行             |
| y + 移动键 | 复制选定区域           |
| p       | 粘贴               |
| "ayy    | 复制当前行到 a 寄存器     |
| "ap     | 从 a 寄存器粘贴        |
| "Ayy    | 追加复制当前行到 a 寄存器   |
| "bd     | 删除当前行并保存到 b 寄存器  |
| :reg    | 查看所有寄存器内容        |

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
### undotree
`：UndotreeToggle`（已映射为`F5`），左边是undo tree 树状结构，可以上下移动、选择历史的某一个更改节点然后`enter`
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
## journalctl日志、eog
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
>些sed只有使用-i后才能修改原文件（但是就不会有stdout了）

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
>SIGHUP不会直接导致进程终止，但进程接收到SIGHUP后一般会自行终止

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
## tmux（终端多路复用器）
> 在tmux的命令行里面按住shift进行选中内容，然后ctrl+shift+c复制内容
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
# 连接到名为“0”的session（alias是t0)

tmux new -s "session 1" -n "window 1"
#  创建名为“session 1”的新session,第一个window名为“window 1”
tmux neww -n "window 2"
# 创建window 2

tmux kill-session -t 0
# 终结session（alias是tk 0)

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
## remote machine
```bash
ssh ustc@192.168.176.129 # 从物理机连上虚拟机，需要输密码
logout                                    # 退出

ssh ustc@192.168.176.129 ls # 在虚拟机（模拟远程机）上使用ls命令

```
> 在物理机的git bash上使用`Shift + Insert`或者鼠标右键粘贴


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

git rm file
# 停止跟踪该文件，同时删除本地文件
git rm --cached file
# 停止跟踪该文件，同时保留本地文件（如果该文件已在暂存区，则也会从暂存区中移除）

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
>建议先多用 `merge`,只对==尚未推送或分享给别人==的==本地==修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作
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
>perf 和 stress用不了

>课后题的最后一题是关于wireshark抓包的
## 调试
### 日志
>日志文件大多位于/var/log/文件夹下
>lnav    一个交互式、彩色的日志查看器

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
#### pdb/ipdb
```bash
python3 -m ipdb bubble.py
# 使用ipdb调试
```

| ipdb命令              | 说明                                                                                           |
| ------------------- | -------------------------------------------------------------------------------------------- |
| `l`                 | 显示当前执行位置附近的源代码（默认显示11行），可以用 `l [start],[end]` 指定显示范围。                                        |
| `s`                 | 单步执行，进入函数内部。                                                                                 |
| `n`                 | 单步执行，但不进入函数内部（函数调用会被视为一个原子操作）。                                                               |
| `c`                 | 继续执行程序直到遇到断点或程序结束。                                                                           |
| `!c`                | 输出c的值（而不是continue)                                                                           |
| `b 6`               | 在第6行设置断点。                                                                                    |
| `tbreak 10`         | 临时断点（命中后自动删除）                                                                                |
| `b`                 | 查看断点                                                                                         |
| `pm()`              | 停在最后一个抛出异常的地方的前面(可以检查在程序出错之前程序的状态)<br>相当于pdb.post_mortem(sys.last_traceback) ，后者需要import sys |
| `r`                 | 继续执行直到当前函数返回。                                                                                |
| `q`                 | 退出调试器，终止程序执行。                                                                                |
| `cl` (clear)        | 清除断点，可以用 `cl [断点号]` 清除指定断点，或者 `cl` 清除所有断点。                                                   |
| `p`                 | 打印表达式的值，例如 `p variable`。                                                                     |
| `pp` (pretty print) | 美化打印表达式的值，更易阅读。                                                                              |
| `p locals()`        | 打印当前作用域内的局部变量字典。                                                                             |
| `restart`           | （ipdb特有）重新启动调试程序，从头开始执行。                                                                     |
| `display`           | (ipdb特有）持续跟踪变量的变化（只在值变动时显示）(undisplay取消)                                                     |
>在python代码中插入`breakpoint()`可以打断点

| 命令  | 全称      | 切换方向   | 作用对象                                    |
| --- | ------- | ------ | --------------------------------------- |
| `w` | `where` | 显示整个栈  | 显示当前的调用栈信息，列出所有调用帧，从最早的调用到当前执行位置（最底到最顶） |
| `u` | `up`    | 向上切换栈帧 | 切换到上层调用者的帧（即“谁调用了当前函数”）                 |
| `d` | `down`  | 向下切换栈帧 | 切换到下层被调用者的帧（即“当前函数调用的函数”）               |
```python
(Pdb) w
> /home/ustc/study/Debugging and Profiling/pdb-tutorial/main.py(13)<module>()
-> main()
  /home/ustc/study/Debugging and Profiling/pdb-tutorial/main.py(9)main()
-> GameRunner.run()
  /home/ustc/study/Debugging and Profiling/pdb-tutorial/dicegame/runner.py(31)run()
-> for die in runner.dice:

# pdb 是反着显示的， 最上面是最底帧，最下面是最上帧
# w命令的结果中， > 标记的是当前帧
```
#### shellcheck的一个例子
```sh
#!/bin/sh
## Example: a typical script with several problems
for f in $(ls *.m3u)                                                                             # 通过ls命令输出来迭代文件名是不可靠的，特别是当文件名包含空格、换行符或其他特殊字符时
												　# 如果文件名以　-　开头，可能会被误解为命令选项
do
  grep -qi hq.*mp3 $f \　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　 # grep模式、变量$f未加引号
    && echo -e 'Playlist $f contains a HQ file in mp3 format'      # echo -e在POSIX shell中未定义； 单引号
done
```

```sh
#!/bin/sh
##  FIXED
for f in ./*.m3u
do
  grep -qi "hq.*mp3" "$f" \
	   && printf "Playlist %s contains a HQ file in mp3 format\n" "$f"
done
```
### 反向调试
>具体工具有`revPDB`(Python),`rr`(C/C++)，不过没亲自动手试过

>传统调试器只能向前执行，而Bug定位需要向后思考
>反向调试器让开发者可以像侦探一样，从错误结果"倒推"到原因，节约时间
### 系统调用
> 系统调用是指“用户程序调用内核”，然后内核再执行功能（如访问硬件）

```bash
strace ls -l   # 跟踪 ls -l 命令的系统调用过程
strace ls -l > stdout.txt 2> stderr.txt
# stdout.txt: ls 的文件列表
# stderr.txt: 系统调用跟踪信息

sudo strace -p <PID> # 追踪某进程的系统调用

strace -e trace=open,read,write ls # 只显示 ls 命令中涉及 open、read 和 write 的系统调用
```
### 静态分析
>英语静态分析工具writegood和proselint(前者需要安装 Node.js 和 npm)

```bash
echo "very unique thing" | proselint
# <stdin>:1:2: uncomparables.misc Comparison of an uncomparable: 'very unique' is not comparable.
# <stdin>:1:2: weasel_words.very Substitute 'damn' every time you're inclined to write 'very'; your editor will delete it and the writing will be just as it should be.
```

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
#### Vim插件neomake
```vim
:Neomake                     # 把shellcheck集成到编辑器中
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
### CPU分析器

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
# -s tottime按tottime排序，从大到小显示。排除子函数时间
# 如果是比较不同函数的性能，推荐使用-s time
# 还可以-o output_file，把分析结果保存到文件

# tottime	函数“自身”耗费的总时间（不包括子函数）
# cumtime	累计时间（函数本身 + 所有子函数调用的时间总和）

#############（另外一次分析的结果实例）################
Ordered by: internal time

   ncalls           tottime  percall  cumtime  percall     filename:lineno(function)
    78454    0.032       0.000    0.078    0.000          random.py:291(randrange)
    78454    0.022       0.000    0.034    0.000          random.py:242(_randbelow_with_getrandbits)
34158/1000    0.016       0.000    0.020    0.000          origin_sorts.py:23(quicksort)
34068/1000    0.013      0.000    0.015    0.000           origin_sorts.py:32(quicksort_inplace)
    78454    0.013       0.000    0.091    0.000           random.py:332(randint)
   235362        0.012       0.000    0.012    0.000          {built-in method _operator.index}
        3    0.011       0.004    0.144    0.048           origin_sorts.py:4(test_sorted)
    99348    0.007       0.000    0.007    0.000           {method 'getrandbits' of '_random.Random' objects}
     1000      0.006       0.000    0.006    0.000            origin_sorts.py:11(insertionsort)
……
```
#### line_profiler（行级别）
```bash
kernprof -l -v urls.py
# urls.py是访问指定网页，解析其 HTML 内容，并提取页面中所有链接的 URL
#　line_profiler是逐行跟踪分析器，要在def函数定义的正上方一行插入装饰器@profile，用于定位函数内部“性能瓶颈”

# Per Hit：每次执行的平均时间
# % Time：该行代码占函数总执行时间的百分比


#######################################
Wrote profile results to urls.py.lprof     # 这个lprof文件要用python3 -m line_profiler打开
Timer unit: 1e-06 s

Total time: 0.474917 s
File: urls.py
Function: get_urls at line 7

Line #      Hits         Time        Per Hit    % Time  Line Contents
==============================================================
     7                                                                 @profile
     8                                                                 def get_urls():
     9         1     465727.0 465727.0     98.1      response = requests.get('https://missing.csail.mit.edu')
    10         1       8781.6    8781.6         1.8        s = BeautifulSoup(response.content, 'lxml')
    11         1          0.3          0.3             0.0       urls = []
    12        48      389.0        8.1             0.1       for url in s.find_all('a'):
    13        47       18.7         0.4             0.0           urls.append(url['href'])

```
### 内存分析器
#### memory_profiler（行级别）
```bash
python3 -m memory_profiler mem.py
# Mem usage	当前行代码执行前的内存使用量（单位是 MiB，约等于 MB）
# Increment	这一行执行后相对上一次内存使用的增量（为 0 表示没有新增占用）
# Occurrences	该行被执行的次数（可用于判断是否在循环中被频繁执行）


######################################
Filename: mem.py

Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     1   22.500 MiB   22.500 MiB         1              @profile
     2                                                                          def my_func():
     3   30.125 MiB    7.625 MiB           1                        a = [1] * (10 ** 6)
     4  182.750 MiB  152.625 MiB        1                        b = [2] * (2 * 10 ** 7)
     5   30.230 MiB -152.520 MiB        1                        del b
     6   30.230 MiB    0.000 MiB           1                        return a
```
### 事件分析器
>perf,报告与程序相关的系统事件

```bash
sudo perf stat stress -c 1
# 使用 stress 工具对系统进行 1 个 CPU 核心的压力测试，并通过 perf 的stat工具收集其运行过程中的性能统计数据
sudo perf record stress -c 1
# 记录程序正在执行的操作
sudo perf report
# 执行 perf record 采样完成后，用 perf report 来查看采样结果
```
### 可视化
#### flame graph (采样分析)
查看程序的栈
#### 调用图
```bash
pycallgraph graphviz -- ./fib.py    # --避免当文件名或者参数以 - 开头时，被误解析成命令行选项
eog pycallgraph.png
```
![[pycallgraph.png]]

### 资源监控
`htop`  (类似的还有glances,dstat)，写脚本用`top`(命令行工具,输出的是文本)

| htop<br>快捷键   | 功能描述                  | 说明与用途                                      |
| ------------- | --------------------- | ------------------------------------------ |
| F5 / t        | 显示进程树状结构              | 展示父子进程关系，观察服务或脚本启动的子进程结构。                  |
| F6 / > / .    | 选择排序方式                | 可以选择按 CPU%、内存占用、PID、运行时间等排序。               |
| N / P / M / T | 快速按 PID、CPU%、内存%、时间排序 | 分别代表 `PID`、`%CPU`、`%MEM`、`TIME+`。快速排序用得很多。 |
| F3 / /        | 按进程名搜索                | 快速定位特定进程，如 `python`、`nginx`。               |
| F4 / \        | 筛选显示进程                | 只显示匹配关键词的进程，适合聚焦查看。                        |
| F9 / k        | 杀死进程                  | 选中进程后按此键，选择信号终止进程。                         |

```bash
df -h                  # 查看整体磁盘使用情况
du -h ~/study   # disk usage
ncdu                  # du的交互版
free -h               # 显示系统当前空闲的内存

python3 -m http.server 4444
lsof | grep ":4444 .LISTEN"
# lsof 可以列出当前系统中所有被打开的文件（类 Unix 系统中，几乎一切都是文件，包括常规文件、目录、套接字、管道、设备等）

hyperfine --warmup 3 'fdfind -e py' 'find . -iname "*.py"'
#　fdfind vs find性能对比（使用 hyperfine）

 iotop
# 可以显示实时 I/O 占用信息而且可以非常方便地检查某个进程是否正在执行大量的磁盘读写操作

ss -tuln                # 查看系统中监听的 TCP 和 UDP 端口

sudo nethogs     # 对网络占用进行监控(还有iftop也可以)
```
# 8.元编程
## 构建系统

### Makefile
**Makefile**：
```makefile
.PHONY: clean all do
# 声明三者为伪目标，只是动作名称，与同名文件无关； 否则会检查比较同名文件文件和它的依赖文件的时间戳，以此决定是否执行相应命令

all: paper.pdf do

paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py                               # %是通配符
	./plot.py -i $*.dat -o $@                        # $* 表示通配符匹配的名字;$@ 表示目标文件的完整名字

do:
	touch do.txt
# make默认执行第一个目标,所以如果没有 all 目标，执行 make 时默认只会构建 paper.pdf，do 不会被执行

clean:
	rm paper.pdf
	rm do.txt
	rm plot-data.png
	rm paper.log
	rm paper.aux
```

**recursive_makefile**：（递归调用子目录）
```makefile
SUBDIRS = foo bar baz

.PHONY: subdirs $(SUBDIRS) clean

subdirs: $(SUBDIRS)

$(SUBDIRS):
	$(MAKE) -C $@

foo: baz         #foo 这个目标在执行前，必须先执行 baz
# 因此即使 subdirs 的依赖写的是 foo-bar-baz，Make 会先根据依赖关系排序 baz-foo-bar

clean:
	rm bar/bar.txt
```

>同一个目录下的两个makefile:使用`make -f recursive_makefile`来执行

>在 Makefile 中，如果把一个伪目标作为某个真实文件的依赖项，那么因为这个伪目标总是被认为“过期”，它的命令每次都会执行，并可能导致真实目标也被误认为“需要重建”

>伪目标不会走隐式规则查找
>隐式规则：例如`make foo.o`，如果Makefile中没有foo.o的规则，就自动尝试执行`gcc -c foo.c -o foo.o`

| 文件名/过程名 | 文件类型              | 说明                                   |
| ------- | ----------------- | ------------------------------------ |
| `foo.c` | 源代码文件             | 用C语言编写的程序源码，是人类可读的文本文件               |
| 编译      |                   | gcc -c foo.c -o foo.o                |
| `foo.o` | 目标文件（Object file） | 编译器生成的机器代码，但未链接，不能直接执行               |
| 链接      |                   | gcc foo.c -o foo<br>gcc foo.o -o foo |
| `foo`   | 可执行文件             | 链接器将目标文件和库文件结合生成的可执行程序，可以运行          |

>`evince`——默认的文档查看器
>`xdg-open`——用系统默认的程序打开（任意类型）

>写Makefile时，应该提供一些标准的目标（target），例如all、install、clean、check、uninstall，方便别人使用你的项目
## 依赖管理
### 语义化版本
1. `主版本号`：如果修改了 API 但是它并不向后兼容
2. `次版本号`：如果添加了 API 并且该改动是向后兼容的
3. `修订号`：没有改变 API,做了向后兼容的问题修正

**版本要求语法**：
- 默认要求（如 `1.2.3` 代表 `>=1.2.3, <2.0.0`）
- 插入符要求（`^1.2.3`，与默认相同）
- 波浪号要求（`~1.2.3` 代表 `>=1.2.3, <1.3.0`）
- 通配符要求（`1.*` 代表 `>=1.0.0, <2.0.0`）
- 比较要求（如 `>= 1.2.0`、`< 2`）

| 写法               | 意义                                | 是否允许升级？                                        | 推荐情况             |
| ---------------- | --------------------------------- | ---------------------------------------------- | ---------------- |
| `foo = "1.2.3"`  | **语义化版本范围**，等价于 `>=1.2.3, <1.3.0` | ✅ 会自动升级到 `<1.3.0` 的 patch 版本，如 `1.2.4`、`1.2.9` | ✅ 推荐，默认写法        |
| `foo = "=1.2.3"` | **精确版本匹配**，只允许使用 `1.2.3` 一个版本     | ❌ 不会升级，其他版本一律不行                                | ⚠️ 特殊情况使用，平时不推荐  |
| `foo = "~1.2.3"` | **波浪号版本匹配**，等价于 `>=1.2.3, <1.3.0` | ✅ 与 `"1.2.3"` 效果一样                             | ❌ 不推荐，意义相同但语法不主流 |

> foo = "1.0"   等价于   foo >= 1.0.0, < 2.0.0 ,但不会匹配 1.0.0-alpha

**锁文件**：列出了当前每个依赖所对应的具体版本号
**vendoring**：把依赖中的所有代码直接拷贝到项目中

## 持续集成
**CI**：当您的代码变动时，自动运行的东西（例如Travis CI、Azure Pipelines 和 GitHub Actions）
>在代码仓库中添加一个文件，描述当前仓库发生任何修改时，应该如何应对(例如：如果有人提交代码，执行测试套件)

### git hook
>.git/hooks/ 目录下的那些 \*.sample 文件是钩子脚本的示例，它们默认被命名为 .sample 后缀，因此不会自动生效，需要你去除后缀并chmod

```bash
# 下面是一个 pre-commit 钩子文件的一部分，会在提交前执行 make paper.pdf 并在出现构建失败的情况拒绝commit

echo "Running make paper.pdf before commit..."
if ! make paper.pdf; then
    echo "Build failed. Commit aborted."
    exit 1
fi

echo "Build succeeded. Proceeding with commit."
exit 0
```

> 这里的1和0与之前的`$?`还有`while kill -0 $1 do …`，都是**返回状态值**，**1**是**失败，0**是**成功**

### Github Actions
- **Workflow（工作流）**：
  每个工作流由一个 `.yml` 文件描述，位于仓库中的 `.github/workflows/` 目录下。
  每个 workflow 可以定义触发条件和执行任务。

- **Job（作业）**：
  工作流中的一组任务，可以并行或串行运行。

- **Step（步骤）**：
  作业中的每一步操作，通常包括运行脚本或使用预定义的 action。

例如：
- `shellcheck.yml`：当 `.sh` 文件被提交时自动运行 `ShellCheck`
- `proselint.yml`：当 `.md` 文件被提交时自动运行 `proselint`
## 测试
- 测试套件（Test suite）：所有测试的统称。
- 单元测试（Unit test）：一种“微型测试”，用于对某个封装的特性进行测试。
- 集成测试（Integration test）：一种“宏观测试”，针对系统的某一大部分进行，测试其不同的特性或组件是否能 _协同_ 工作。
- 回归测试（Regression test）：一种实现特定模式的测试，用于保证之前引起问题的 bug 不会再次出现。
- 模拟（Mocking）: 使用一个假的实现来替换函数、模块或类型，屏蔽那些和测试不相关的内容。例如，您可能会“模拟网络连接” 或 “模拟硬盘”。
# 9.安全和密码学
## 熵
$$
H = L \times \log_2 N
$$
- $H$ 表示熵（单位：bit）
- $L$ 表示密码长度
- $N$ 表示可能字符的种类数量
## 哈希函数/散列函数
```bash
printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
```

- 确定性：对于不变的输入永远有相同的输出。
- 不可逆性：对于 $hash(m) = h$，难以通过已知的输出 $h$ 来计算出原始输入 $m$。
- 目标碰撞抵抗性/弱无碰撞：对于一个给定输入 $m_1$，难以找到 $m_2$ != $m_1$ 且 $hash(m_1) = hash(m_2)$。
- 碰撞抵抗性/强无碰撞：难以找到一组满足 $hash(m_1) = hash(m_2)$ 的输入 $m_1$, $m_2$`（该性质严格强于目标碰撞抵抗性）。
> sha1sum用SHA-1，sha256sum用SHA-2
### 应用
- `内容寻址存储`（如Git）（与位置寻址等相对）
- `哈希校验`：从非官方镜像站下载ISO文件验证是否内容被篡改
- `承诺机制`：（脑海中抛硬币）我随机选择一个值 r，并和你分享它的哈希值 h = sha256(r)。 这时你可以开始猜硬币的正反：我们一致同意偶数 r 代表正面，奇数 r 代表反面。 你猜完了以后，我告诉你值 r 的内容，得出胜负。同时你可以使用 sha256(r) 来检查我分享的哈希值 h 以确认我没有作弊
## 密钥生成函数KDF
> 通常较慢：防止被暴力破解

> 问题：假设两个用户都用了 123456 作为密码，如果没有加`盐`，他们的哈希值完全一样，攻击者会立刻知道哪些人用了相同密码，以及同一个用户在不同的平台使用同一个密码的情况
> 防止`彩虹表`攻击：`KDF(password + salt)`
> 在验证登录请求时，使用输入的密码连接存储的盐重新计算哈希值 `KDF(input + salt)`，并与存储的哈希值对比

## 对称加密
> 代表：`AES`

```text
keygen()   ->   key (这是一个随机方法)
encrypt(plaintext, key)   ->   ciphertext (输出密文)
decrypt(ciphertext, key)   ->   plaintext (输出明文)
```

> 可用于把文件上传至Dropbox等云平台时的加密

> 用户不用直接记住复杂的key密钥，而是记住passphrase密码,然后key = KDF(passphrase, salt)，并且密钥是随机生成的二进制数据，直接存储或传输风险较大

```bash
openssl aes-256-cbc -salt -in test.md -out test.enc.md
openssl aes-256-cbc -d -in test.enc.md -out test.dec.md

cmp test.dec.md test.md 
# 无输出
echo $?
# 0
```
> cmp输出简洁，通常是“文件名 第N字节不同”，用于快速检测文件是否完全相同

## 非对称加密
> 代表：`RSA`

```text
keygen()   ->   (public key, private key)  (这是一个随机方法)

encrypt(plaintext, public key)   ->   ciphertext  (输出密文)
decrypt(ciphertext, private key)   ->   plaintext  (输出明文)

# 签名和验证
sign(message, private key)   ->   signature  (生成签名)
verify(message, signature, public key)   ->   bool  (验证签名是否是由和这个公钥相关的私钥生成的，bool代表True or False)
```
### 应用
- PGP电子邮件加密
- 聊天加密（例如Signal、Telegram)
- 签署软件发布
- Git Tag有签名
- 挑战应答机制：服务器生成一个随机挑战（随机数/随机字符串），发给客户端——客户端使用私钥对这个挑战进行数字签名，并将签名返回给服务器——服务器用存储的公钥验证签名的有效性

### PGP加密邮件

**第一步**：导入对方的公钥  
```bash
curl https://keybase.io/<username>/pgp_keys.asc | gpg --import
````

- `<username>` 替换为对方的 Keybase 用户名
    
- 这会将对方的 **PGP公钥** 导入到本地 GPG 密钥环中

**第二步**：验证对方的身份指纹

```bash
gpg --edit-key "<real name or email>"
```

在 `gpg>` 控制台中输入：`fpr`，记录下输出的 **主密钥指纹**，并与对方在 Keybase、GitHub 或官网上发布的指纹进行比对。  
示例输出：

```
主密钥指纹： 72EE 4824 FA6E FF1F E750  A015 C3F6 E4F5 086B 3B32
```

如果一致，说明该 **公钥** 可信，可以安全使用。

**第三步**：编写你的邮件内容，例如 `message.txt`

**第四步**：加密并签名消息

```bash
gpg --encrypt --sign --armor -r <recipient-email-or-keyid> message.txt
```
- `--encrypt`：用对方的 **公钥** 加密消息
- `--sign`：用你自己的 **私钥** 对消息签名
- `--armor`：输出 ASCII 格式（适合用邮件发送）
- `-r`：指定前面 `--encrypt` 使用的 **公钥** 所属的接收人邮箱或 GPG Key ID（需与公钥一致）
> 成功后将生成 `message.txt.asc` 文件

**第五步**：通过邮件客户端（例如 Gmail）发送加密内容

- 将 `message.txt.asc` 的内容复制到正文，或作为附件发送
- 邮件主题建议使用明文，例如：`Confidential Message`


> 如果你希望对方能加密回复你，可以用下面这个命令生成你的公钥文件，一起发送给对方：
```bash
gpg --armor --export yourself@gmail.com > mykey.asc
```

**其他常用命令**：

```bash
gpg --list-keys   # 列出你本地保存的所有公钥
```

```text
# 示例
pub   rsa4096 2014-10-30 [SC]
      C3F6E4F5086B3B32
uid           [ unknown] Anish Athalye <me@anishathalye.com>
```
> 其中 `C3F6E4F5086B3B32` 就是 Key ID

## 密钥分发
1. PKI:信任一个权威的`证书颁发机构CA`，然后通过它颁发的数字证书，来验证某个公钥确实属于某人
2. PGP使用`信用网络`:我信任张三签署的密钥，那么张三签署的李四的密钥我也可以一定程度上信任
3. Signal使用`TOFU`：信任用户第一次使用时给出的身份，此后严密监控密钥是否变动
4. Keybase使用的`设备信任链机制`：新设备加入账户时，必须由现有已信任设备签名授权，多个设备共同维护账户密钥链，预防“中间人攻击”
5. Keybase使用的`social proof`：绑定其他账号
## 其他
密码管理器、2FA、全盘加密、`git commit -S`(Git 中用来对提交进行 GPG 签名) 、`git tag -s`和`git tag -v`

# 10.大杂烩
## AutoHotkey
## 守护进程deamon
```bash
systemctl status # 查看正在运行的所有守护进程
systemctl status NetworkManager #　进一步查看具体守护进程

crontab -e 
# 打开vim界面后在最下方写,每周一10点定时更新vim插件
0 10 * * 1 ~/scripts/Vim_plug_upgrade.sh
0 * * * * top -b -n1 >> ~/logs/cpu.log
```
> cron是定时任务调度器

> 用户自定义服务配置文件存在`/etc/systemd/system/`目录下。所有位于这个目录的服务文件都可以被 systemctl 命令管理（enable（启用）、disable（禁用）、start（启动）、stop（停止）、restart（重启）、或者 status（检查）等），而且你可以自己新建service文件（配置systemd)
```bash
sudo systemctl daemon-reload # 新建或修改 .service 文件后的步骤
sudo systemctl enable tmux.service # 自启动
```

## Fuse
> 传统流程：touch hello.txt -> 用户态 -> 内核态 -> VFS -> EXT4
> FUSE流程：touch hello.txt -> 用户态 -> 内核态 -> VFS -> FUSE内核模块（fuse.ko) -> FUSE 守护进程(用户态)

> `touch hello.txt`实际上,touch（用户层面程序）并不直接操作硬盘,它只是调用了一个`系统调用`（比如 open() 或 creat()）,“系统调用”是程序与内核交流的唯一方式

> **FUSE**（Filesystem in Userspace/用户空间文件系统）允许运行在用户空间上的程序实现文件系统调用，并将这些调用与内核接口联系起来。

例子：
**rclone**:提供了一种像本地一样访问远程云端空间（如dropbox、google drive)的方式，因此在实际使用中可以起到“扩容”的效果
**sshfs**:使用 SSH 连接在本地打开远程主机上的文件
**gocryptfs**:文件以加密形式保存在磁盘里，但挂载gocryptfs后用户可以直接从挂载点访问文件的明文
还有kbfs、borgbackup等

## 备份
## API
```bash
curl https://api.weather.gov/points/42.3604,-71.094 > weather_data.json
curl $(jq -r '.properties.forecastHourly' weather_data.json)
```

> `json`文件（JavaScript Object Notation）本质上是纯文本文件，常用于数据的存储与传输，特别是前后端通信、配置文件、API

> `jq`是专门用来处理 JSON 数据的命令行工具

> 有些需要认证的 API 通常要求用户在请求中加入某种私密令牌(相关：OAuth)

## 命令行标志参数
> git push --dry-run (“空运行”，可以确认工具真实运行时会进行的操作)

```bash
ssh -v user@host # 输出基本的调试信息

ssh -vv user@host # 输出更详细的调试信息

ssh -vvv user@host # 输出最详细的调试信息
```

> `--quiet`：正常的输出信息都会被隐藏，只有当出现错误时，才会显示错误提示

```bash
#  "-"作为文件名的占位符时，代表不使用具体的文件，而是从标准输入（键盘输入或管道传入的数据）读取内容，或者将结果输出到标准输出

echo "hello world" | grep "hello" -

tar -czf archive.tar.gz .
tar -cf - . | gzip > archive.tar.gz
# 二者等效
```

```bash
ssh machine --for-ssh -- foo --for-foo
# "machine"是指远程服务器的名字
# for-ssh是给ssh命令的标志参数
# for-foo是给foo命令的标志参数（foo命令是你要在machine上运行的一个程序）
# "--"的作用就是防止for-foo被误认为是ssh的标志参数
```

## 平铺式（tiling）管理器（瓦片式）
## Markdown
[超链接](https://cs-trekker.github.io/)
**\<br\>** 实现表格内换行
## Hammerspoon (macOS 桌面自动化)
## live USB
1. 安装操作系统
通过将操作系统镜像写入 USB 闪存盘，制作成启动盘，可以用来在计算机上安装新的操作系统。这是最常见的用途。
2. 无需安装，直接运行操作系统
用户可以通过 Live USB 直接启动和运行完整的操作系统，而不必将其安装到硬盘。（临时使用、测试操作系统或者在不破坏现有系统）
3. 系统修复与维护
如果计算机的硬盘操作系统出现故障，无法正常启动，Live USB 可以用来启动计算机，进行系统修复操作
4. 数据恢复
当硬盘系统损坏或数据丢失时，可以使用 Live USB 启动系统，访问硬盘数据，进行备份或恢复，避免数据进一步损坏。

## 虚拟机、Vagrant、Docker、OpenStack
## Jupyter Notebook
win+R -> cmd -> 输入`jupyter notebook` -> 浏览器
# Q&A
- `/bin` - 基本的系统工具
- `/usr/` - 只读的用户数据
    - `/usr/bin` - 非必须的命令二进制文件（存放用户程序）
    - `/usr/sbin` - 非必须的系统二进制文件，通常是由 root 运行的
    - `/usr/local/bin` - 用户编译程序的二进制文件

> 在你的系统中，`/bin`是`/usr/bin`的符号链接

- `/dev` - 设备文件，通常是硬件设备接口文件
- `/etc` - 主机特定的系统配置文件
- `/home` - 系统用户的主目录
- `/lib` - 系统软件通用库
- `/opt` - 可选的应用软件
- `/sys` - 包含系统的信息和配置
- `/tmp` - 临时文件( `/var/tmp` ) 通常重启时删除
- `/var` -变量文件（随着时间而变化） 像日志或缓存

> `column -t`：按空格分列并对齐

> `pandoc`：格式转换(md、html、pdf、LaTeX、docx……)


浏览器插件：
- `stylus`自定义和修改网站的外观/CSS样式
- `Firefox Multi`允许用户将浏览器中的 Cookie、缓存及其他网站数据划分到不同的“容器”中，从而实现以不同身份独立浏览多个网页，增强隐私保护
- `密码集成管理器`与浏览器集成的好处是自动监测钓鱼网站域名


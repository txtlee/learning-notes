# 第10章 认识与学习BASH

# 第 10 章 · 认识与学习 BASH

> 核心一句话:**BASH 是你和 Linux 内核对话的"翻译官",学会它 = 把命令行从"敲命令"升级为"自动化操作"。** 本章四大块:① Shell 与 BASH 简介 ② 变量(核心)③ 数据流重定向 ④ 管线与文本处理。

---

## 一、Shell 与 BASH 简介

### 1. 什么是 Shell

```
用户 → 【Shell】 → 内核(Kernel) → 硬件
```
- **Shell**:命令行解释器,把用户输入的命令翻译给内核执行。
- **BASH**:Linux 默认的 Shell(Bourne Again SHell),最常用。

### 2. 系统有哪些 Shell

```bash
cat /etc/shells      # 查看系统支持的合法 shell
echo $SHELL          # 查看当前默认 shell
```
常见:/bin/bash、/bin/sh、/bin/zsh 等。

### 3. BASH 的六大优点

| 特性 | 说明 | 示例 |
|------|------|------|
| **命令记忆** | history 记录用过的命令 | `history`、`!!`、`!n` |
| **补全功能** | Tab 自动补全命令/文件/参数 | 输入 `ls /et[Tab]` → `ls /etc/` |
| **命令别名** | alias 自定义命令快捷方式 | `alias ll='ls -al'` |  `alias c='clear'`
| **作业控制** | 前台/后台、暂停恢复 | `ctrl+z`、`jobs`、`fg`、`bg` |
| **脚本** | 可把命令写成程序 | shell script |
| **通配符** | 简化文件名匹配 | `ls *.txt` |

### 4. type:查询命令来源

```bash
type ls          # ls is aliased to 'ls --color=auto'
type cd          # cd is a shell builtin       ← 内置命令
type /bin/ls     # /bin/ls is /bin/ls           ← 外部命令(文件)
```
- **内置命令**:bash 自带,如 `cd`、`pwd`、`echo`,执行快。
- **外部命令**:独立程序文件,存在 PATH 目录里,如 `ls`、`grep`。

---

## 二、Shell 的变量(本章核心)

### 1. 变量的取用与设置

```bash
echo $var            # 取用变量,要加 $
var=value            # 设置变量,等号两边不能有空格!
myname="Zhang San"   # 含空格用引号
注意，我试了下，如果是用内置的方法，=后要跟一个空格：
    now= date ｜ 输出的是日期
    now=date ｜ 输出的是date字符串
```

### 2. ⭐ 变量设置规则(重点,易错)

| 规则 | 说明 | 正确 / 错误 |
|------|------|-------------|
| 等号两边不能有空格 | `var = value` 会出错 | ✅ `var=value` ❌ `var = value` |
| 变量名只能字母数字下划线 | 不能数字开头 | ✅ `var_1` ❌ `1var` |
| 双引号保留特殊字符 | `$var` 会展开 | `"hello $name"` |
| 单引号纯文本 | `$var` 不展开 | `'hello $name'` → 字面量，即单引号输出'$name'这个字符串，而不输出name的实际值
| 转义符 `\` | 取消特殊含义 | `\$` → 字面的 $ |
| 命令替换 | 反单引号或 `$()` | `now=$(date +%F)` |
| 扩展为环境变量 | `export var` | 子进程才能用 |
| 取消变量 | `unset var` | |

**双引号 vs 单引号(重点示例):**
```bash
name="Tom"
echo "Hi $name"     # Hi Tom      ← 双引号会展开变量
echo 'Hi $name'     # Hi $name    ← 单引号原样输出
```

**命令替换示例:**
```bash
# 把命令结果存进变量
now=$(date +%F)          # 推荐 $() 写法
now2=`date +%F`          # 反单引号也行,但易混淆
echo "今天是 $now"       # 今天是 2026-07-01

# 嵌套:列出当前目录文件数
count=$(ls | wc -l)
```

### 3. 环境变量 vs 自定义变量

| 类型 | 特点 | 查看 |
|------|------|------|
| **环境变量**(大写) | 全局,子进程继承 | `env` |
| **自定义变量**(小写) | 局部,仅当前 shell | `set`(看全部) |

```bash
env                  # 只看环境变量
set                  # 看所有变量(含环境+自定义+函数)
export myvar         # 把自定义变量升级为环境变量
```

常见环境变量:`$HOME`(家目录)、`$PATH`(命令搜索路径)、`$USER`、`$SHELL`、`$LANG`、`$PWD`。

### 4. read:从键盘读入变量

```bash
read -p "请输入姓名: " name    # -p 提示语
echo "你好, $name"

read -t 5 -p "5秒内输入: " ans  # -t 超时5秒
```

### 5. 数组 array

```bash
arr=(apple banana cherry)
echo ${arr[0]}        # apple(下标从0)
echo ${arr[@]}        # apple banana cherry(全部)
echo ${#arr[@]}       # 3(元素个数)
```

### 6. declare / typeset:声明变量类型

```bash
declare -i num=10+5   # -i 声明整数,可运算 → 15
declare -x var        # -x 等同 export
declare -r PI=3.14    # -r 只读,不可改
declare -a arr        # -a 数组
```

### 7. ulimit:限制用户资源

```bash
ulimit -a             # 查看所有限制
ulimit -f 10240       # 限制可创建文件最大 10MB
```

---

## 三、⭐ 变量内容的高级操作(重点,必会示例)

### 1. 变量内容的删除

```bash
path=/usr/local/share/doc

# 从【变量开头】删除匹配(# 短匹配, ## 长匹配)
echo ${path#*/}       # usr/local/share/doc   ←删最前面的 /
echo ${path##*/}      # doc                    ←删到最后一个 /(取文件名,同 basename)

# 从【变量末尾】删除匹配(% 短匹配, %% 长匹配)
echo ${path%/*}       # /usr/local/share       ←删末尾 /doc(取目录,同 dirname)
echo ${path%%/*}      # (空)                   ←从末尾最长匹配 /*,全删
```

记忆:**`#` 在键盘 `$` 右边靠前 → 从前面删;`%` 在右边靠后 → 从后面删。一个符号短匹配,两个长匹配。**

### 2. 变量内容的替换

```bash
str="hello world hello"

echo ${str/hello/HI}    # HI world hello     ← / 替换第一个
echo ${str//hello/HI}   # HI world HI        ← // 替换全部
```

### 3. 变量的测试与默认值

```bash
# 假设 var 未设置或为空

echo ${var-abc}     # var未设置时使用默认值abc，如果var已设置为空字符串""，则结果为空
echo ${var:-abc}    # var未设置或为空字符串使用默认值abc
echo ${var=abc}     # 同 :- ,但同时把 abc 赋给 var
echo ${var?err}     # var 未设置 → 输出 err 并退出
```

**实际场景:给变量设默认值**
```bash
# PORT 没设就用 8080
PORT=${PORT:-8080}
echo "服务端口: $PORT"
```

---

## 四、命令别名与历史命令

### 1. alias / unalias

```bash
alias                # 查看所有别名
alias ll='ls -al'    # 设置别名
alias rm='rm -i'     # 让 rm 默认询问(安全)

unalias ll           # 取消别名
```
> 别名只在当前 shell 有效;想永久生效要写进 `~/.bashrc`。

### 2. history 历史命令

```bash
history             # 列出历史命令(带编号)
history 10          # 最近 10 条
!!                  # 执行上一条命令
!n                  # 执行第 n 条命令
!ls                 # 执行最近一条以 ls 开头的命令
```

---

## 五、⭐ 数据流重定向(重点,示例丰富)

### 1. 三种标准数据流

| 名称 | 代号 | 默认 | 含义 |
|------|------|------|------|
| 标准输入 stdin | **0** | 键盘 | 程序读入的数据 |
| 标准输出 stdout | **1** | 屏幕 | 程序正常输出 |
| 标准错误 stderr | **2** | 屏幕 | 程序错误信息 |

### 2. 输出重定向

```bash
ls > list.txt          # stdout 覆盖写入文件
ls >> list.txt         # stdout 追加到文件
ls 2> err.txt          # 执行ls命令，并将其产生的“错误信息”（标准错误）重定向保存到文件err.txt
ls > out.txt 2> err.txt   # stdout 和 stderr 分别写不同文件
ls > all.txt 2>&1      # ⭐ 合并 stdout 和 stderr 到同一文件(经典写法)
ls &> all.txt          # 合并的简写(bash 4+)
ls 2> /dev/null        # 丢弃错误信息(/dev/null 是黑洞)
建议始终使用 2>&1 或 &> 来同时捕获正确输入和错误信息
这里补充一个常用命令：
    nohup python -u code.py > ../log/data_log.log 2>&1 &
    nohup：让程序在用户退出终端后继续运行
    python -u code.p：以非缓冲模式运行Python脚本
    > ../log/data_log.log：将标准输出（stdout）重定向到日志文件
    2>&1：将标准错误（stderr）重定向到与标准输出相同的位置（即也写入该日志文件）
    &：第二个 &，将当前命令作为一个后台作业执行。不加到话命令会在前台运行，阻塞当前终端，直到程序结束。加上后，命令立即返回提示符，可以继续在同一个终端执行其他命令。
```

### 3. 输入重定向

```bash
cat < file.txt         # 从文件读入(代替键盘)
cat << EOF             # 连续输入,直到遇到 EOF 结束
> 第一行
> 第二行
> EOF
```

**实战:用 << 生成配置文件**
```bash
cat > /tmp/test.conf << EOF
host=localhost
port=3306
EOF
```

### 4. 命令执行的判断依据

```bash
cmd1 ; cmd2            # 顺序执行,无论 cmd1 成败
cmd1 && cmd2           # cmd1 成功(返回0)才执行 cmd2
cmd1 || cmd2           # cmd1 失败才执行 cmd2

# 经典用法:目录不存在才创建
mkdir mydir && echo "创建成功" || echo "创建失败"
```

---

## 六、管线命令(Pipeline)与文本处理工具

### 1. 管线 `|`:把前一个命令的输出当后一个的输入

```bash
ls -l | more              # 分页查看
ls -l | grep "txt" | wc -l   # 数 txt 文件个数
这里补充一个常用命令：
    ps -ef | grep python
    ps：Process Status，用于显示当前系统的进程状态
    -e：select all process（选择所有进程）
    -f：full-format listing（以完整格式列出），通常会显示uid,pid,ppid等详细信息
    |：管道符，将前一个命令的输出作为后一个命令的输入
    grep：global regular expression print，用于在文本中搜索匹配特定模式的行
    python：搜索关键词，即只保留包含“python”字符串的行
```

> ⚠️ 管线只传递 **stdout**,stderr 不传(需用 `2>&1` 先合并)。

### 2. 常用文本处理工具(重点,带示例)

#### cut:切分提取字段
```bash
cut -d: -f1 /etc/passwd          # 以:分隔,取第1字段(用户名)
echo $PATH | cut -d: -f3         # 取第3个路径
cut -c1-5 /etc/passwd            # 取每行第1-5个字符
```

#### grep:行过滤
```bash
grep "root" /etc/passwd          # 找含 root 的行
grep -n "bash" /etc/passwd       # -n 显示行号
grep -i "ROOT" /etc/passwd       # -i 忽略大小写
grep -v "nologin" /etc/passwd    # -v 反向(不含 nologin 的行)
grep -c "bash" /etc/passwd       # -c 只数匹配行数
```

#### sort:排序
```bash
sort /etc/passwd                 # 默认按字符排序
sort -t: -k3 -n /etc/passwd      # -t分隔 -k3按第3字段 -n按数字
sort -r file                     # -r 逆序
```

#### uniq:去重(需先 sort)
```bash
sort file | uniq                 # 去重
sort file | uniq -c              # -c 统计每行出现次数
```

#### wc:计数
```bash
wc /etc/passwd                   # 行数 单词数 字节数
wc -l /etc/passwd                # -l 只数行
wc -w file                       # -w 只数单词
```

#### tee:双向分流(既输出到屏幕又存文件)
```bash
ls | tee out.txt                 # 屏幕显示 + 存到 out.txt
ls | tee -a out.txt              # -a 追加
```

#### tr:字符替换/删除
```bash
echo "HELLO" | tr 'A-Z' 'a-z'    # 大写转小写 → hello
echo "a b c" | tr ' ' '-'        # 空格变 - → a-b-c
echo "abc123" | tr -d '0-9'      # -d 删除数字 → abc
```

#### xargs:把输入当参数传给后续命令(重点!)
```bash
# find 找出的结果,作为参数传给 rm
find /tmp -name "*.log" | xargs rm

# 每行转成参数,配合其他命令
cat urls.txt | xargs curl
```

### 3. 管线常用组合示例

```bash
# 统计每种登录 shell 的人数
cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn

# 找出 /etc 下最大的 5 个文件
du -ah /etc | sort -rh | head -5

# 查看占用内存最多的 5 个进程
ps aux | sort -k4 -rn | head -5
```

---

## 七、BASH 的环境配置文件

### 1. login shell vs non-login shell

| 类型 | 触发场景 | 读取配置 |
|------|----------|----------|
| **login shell** | 输入账号密码登录(如 SSH 登录) | `/etc/profile` → `~/.bash_profile` |
| **non-login shell** | 在已登录环境开新终端 | `~/.bashrc` |

### 2. 关键配置文件

| 文件 | 作用 |
|------|------|
| `/etc/profile` | 系统全局,所有用户登录读取 |
| `~/.bash_profile` | 用户个人,login shell 读取(通常内部会调 ~/.bashrc) |
| `~/.bashrc` | 用户个人,non-login shell 读取,**别名和函数常放这** |
| `~/.bash_logout` | 登出时执行 |

### 3. source / `.`:重新加载配置(改完配置不重启生效)

```bash
source ~/.bashrc       # 改完别名后立即生效
. ~/.bashrc            # . 等价于 source
```

### 4. 进站信息

```bash
cat /etc/issue         # 终端登录前的提示信息
cat /etc/motd          # 登录成功后的欢迎信息
```

---

## 八、重点速记清单

1. **BASH 六大特性**:history / Tab补全 / alias / 作业控制 / 脚本 / 通配符。
2. **变量规则**:等号两边不能空格;双引号展开、单引号不展开;`$()` 命令替换。
3. **环境变量**:`env` 看环境,`set` 看全部,`export` 升级为环境变量。
4. **变量删除**:`#`/`##` 从前删,`%`/`%%` 从后删;一个短两个长。
5. **变量替换**:`/`换第一个,`//`换全部。
6. **变量默认值**:`${var:-default}` 最常用。
7. **重定向**:stdout=1、stderr=2;`> 覆盖` `>> 追加`;`2>&1` 合并;`/dev/null` 黑洞。
8. **命令判断**:`;` 顺序、`&&` 成功才后、`||` 失败才后。
9. **管线**:`|` 传 stdout;配合 cut/grep/sort/uniq/wc/tee/tr/xargs。
10. **配置文件**:login shell 读 `~/.bash_profile`,non-login 读 `~/.bashrc`;改完用 `source` 生效。

---

## 九、常见易错点回顾

- ❌ 变量设置 `var = value`(带空格)→ 报错,必须 `var=value`。
- ❌ 单引号里写 `$var` 以为会展开 → 单引号是纯文本,要用双引号。
- ❌ `ls > file 2> file` 想合并 → 两个流同时写会错乱,用 `> file 2>&1` 或 `&> file`。
- ❌ 忘了 `2>&1` 顺序 → 必须写在 `>` 之后,如 `ls > f 2>&1`,不能写 `ls 2>&1 > f`(后者 stderr 仍到屏幕)。
- ❌ `uniq` 不先 `sort` → 去重无效,uniq 只去相邻重复行。
- ❌ 改了 `~/.bashrc` 没生效 → 没用 `source`,或改错了文件(login/non-login 读的不一样)。
- ❌ `${path##*/}` 和 `${path#*/}` 搞混 → 一个取文件名、一个只删第一个 `/`,差别巨大。
- ❌ 管线想传 stderr → 默认只传 stdout,要先 `2>&1` 合并。

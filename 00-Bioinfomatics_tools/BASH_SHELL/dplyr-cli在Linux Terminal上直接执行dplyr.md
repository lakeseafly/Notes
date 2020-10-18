# dplyr-cli：在Linux Terminal上直接执行dplyr

标签（空格分隔）： 生信小工具

---

熟悉R的朋友都会知道，`dplyr`包是对原始的数据集进行清洗、整理以及变换的有力武器之一。但是其使用会局限于你需要有打开R/R studio或者通过R脚本来执行`dplyr`。对于这个问题，今天即将需要介绍的`dplyr-cli`就能很好的解决这个问题。

### dplyr包的介绍

首先再和大家简单介绍一下`dplyr`包（避免有些刚入门的朋友可能不熟悉）。

`dplyr`包是 Hadley Wickham （`ggplot2`包，各种R语言书籍的作者，被称作“一个改变R的人”）的杰作, 并自称 a grammar of data manipulation, 他将原本`plyr` 包中的`ddply()`等函数进一步分离强化,专注接受dataframe对象, 大幅提高了速度, 并且提供了更稳健的与其它数据库对象间的接口。

`dplyr`包的功能主要包括：

 - 变量筛选函数 `select`
 - 筛选函数 `filter`
 - 排序函数 `arrange`
 - 变形（计算）函数 `mutate`
 - 汇总函数 `summarize`
 - 分组函数 `group_by`
 - 多步操作连接符 `%>%`
 - 随机抽样函数 `sample_n,sample_frac`


### dplyr-cli的介绍

了解完`dplyr`包之后，就要介绍咱们这个推文的主角了`dplyr-cli`。`dplyr-cli`设计的初衷就是让我们能够方便快速的在不打开R的情况下，在命令行中运行`dplyr`，处理csv的文件。

目前`dply-cli`支持任何形式的`dplyr::verb(.data, code)`或者`dplyr::*_join(.data, .rhs)`命令。另外支持两个额外的命令，它们并不是原始`dplyr`R包的一部分。

 - csv 不执行dplyr命令，仅将输入数据作为CSV输出到stdout
 - kable不执行dplyr命令，而仅将输入数据作为 `knitr::kable()`格式字符串输出到stdout

其工作原理：`dplyr-cli`使用`{ littler }`在终端中的CSV文件上运行dplyr命令。

> 什么是littler？
littler命令行前端由“ r”（又称“轻量”）提供，作为围绕GNU R语言和统计计算和图形环境的轻量级二进制包装器。尽管R可以在批处理模式下使用，但r二进制文件完全支持'shebang'样式的脚本（即在脚本的第一行中使用hash-mark-exclamation-path表达式）以及在标准Unix管道。**换句话说，该工具提供了无环境的R语言。**


另外一个很友善的功能是，`dplyr-cli`使用终端管道`|`运行命令。


目前的不足：

 - 仅在`OSX`和`YMMV`的bash下测试过
 - 每个命令的实质是在单独的R中运行
 
### 安装

虽然`dply-cli`是可以直接在命令行中直接使用，但是其执行时候还是会依赖到R包。所以首先让我们先把他们安装好：

```
install.packages('littler')  # Scripting front-end for R
install.packages('glue')     # text manipulation
install.packages('dplyr')    # data manipulation
install.packages('docopt')   # CLI description language
```

安装好R包后就是到Github中将`dply-cli`将其shell脚本下载好，并且放到合适的path中来执行：
```
git clone https://github.com/coolbutuseless/dplyr-cli
cp dplyr-cli/dplyr ./somewhere/in/your/search/path
```
  

### 工具使用

安装好工具第一件事当然是看看帮助文档：

```
./dplyr --help

Warning message:
package ‘dplyr’ was built under R version 3.5.2
dplyr-cli

Usage:
    dplyr <command> [--file=fn] [--csv | -c] [--verbose | -v] [<code>...]
    dplyr -h | --help

Options:
    -h --help            show this help text
    -f FILE --file=FILE  input CSV or RDS filename. If reading from stdin, assumes CSV [default: stdin]
    -c --csv             write output to stdout in CSV format (instead of default RDS file)
    -v --verbose         be verbose
```

和R一样，帮助文档首先告诉你当前的 ‘dplyr’的版本，然后一系列执行的参数。


接着我们就通过一系列的实战例子来了解一下如何使用这个好用的工具，这里会使用到`mtcars.csv`这个文件，当你从Github下载`dplyr-cli`时，会包含其作为一个测试文件：

**例子一：简单的基本操作**

输出mpg值为21的行：

```
##这里的 -c选项是用于输出格式为CSV的stdout
cat mtcars.csv | ./dplyr filter -c "mpg == 21"

###输出
"mpg","cyl","disp","hp","drat","wt","qsec","vs","am","gear","carb"
21,6,160,110,3.9,2.62,16.46,0,1,4,4
21,6,160,110,3.9,2.875,17.02,0,1,4,4
```

再让我试一试输出mpg的值小于11的行：

```
cat mtcars.csv | ./dplyr filter -c "mpg < 11"

###结果
"mpg","cyl","disp","hp","drat","wt","qsec","vs","am","gear","carb"
10.4,8,472,205,2.93,5.25,17.98,0,0,3,4
10.4,8,460,215,3,5.424,17.82,0,0,3,4
```

选择名为`cyl`的例，并输出前6行：

```
./dplyr select --file mtcars.csv -c cyl | head -n 6
```

**实例二多个数据处理的参数的结合**

创建名为`cyl2`的新一列，它的值为`cyl`的两倍，再提取`cyl`值为8的行，最后使用`kable`参数，在terminal输出类似表格的结果

```
cat mtcars.csv | \
   ./dplyr mutate "cyl2 = 2 * cyl"  | \
   ./dplyr filter "cyl == 8" | \
   ./dplyr kable
   
###输出

|  mpg| cyl|  disp|  hp| drat|    wt|  qsec| vs| am| gear| carb| cyl2|
|----:|---:|-----:|---:|----:|-----:|-----:|--:|--:|----:|----:|----:|
| 18.7|   8| 360.0| 175| 3.15| 3.440| 17.02|  0|  0|    3|    2|   16|
| 14.3|   8| 360.0| 245| 3.21| 3.570| 15.84|  0|  0|    3|    4|   16|
| 16.4|   8| 275.8| 180| 3.07| 4.070| 17.40|  0|  0|    3|    3|   16|
| 17.3|   8| 275.8| 180| 3.07| 3.730| 17.60|  0|  0|    3|    3|   16|
| 15.2|   8| 275.8| 180| 3.07| 3.780| 18.00|  0|  0|    3|    3|   16|
| 10.4|   8| 472.0| 205| 2.93| 5.250| 17.98|  0|  0|    3|    4|   16|
| 10.4|   8| 460.0| 215| 3.00| 5.424| 17.82|  0|  0|    3|    4|   16|
| 14.7|   8| 440.0| 230| 3.23| 5.345| 17.42|  0|  0|    3|    4|   16|
| 15.5|   8| 318.0| 150| 2.76| 3.520| 16.87|  0|  0|    3|    2|   16|
| 15.2|   8| 304.0| 150| 3.15| 3.435| 17.30|  0|  0|    3|    2|   16|
| 13.3|   8| 350.0| 245| 3.73| 3.840| 15.41|  0|  0|    3|    4|   16|
| 19.2|   8| 400.0| 175| 3.08| 3.845| 17.05|  0|  0|    3|    2|   16|
| 15.8|   8| 351.0| 264| 4.22| 3.170| 14.50|  0|  1|    5|    4|   16|
| 15.0|   8| 301.0| 335| 3.54| 3.570| 14.60|  0|  1|    5|    8|   16|
```

**实例三设置aliases**

通过设置aliases可以进一步方便我们调用这个工具，加快文件处理的速度。将下面的alias放到你.bashrc中：

```
alias mutate="dplyr mutate"
alias filter="dplyr filter"
alias select="dplyr select"
alias summarise="dplyr summarise"
alias group_by="dplyr group_by"
alias ungroup="dplyr ungroup"
alias count="dplyr count"
alias arrange="dplyr arrange"
alias kable="dplyr kable"
```

下面就来体验一下起飞的感觉：

```
cat mtcars.csv | group_by cyl | summarise "mpg = mean(mpg)" | kable

###结果
| cyl|      mpg|
|---:|--------:|
|   4| 26.66364|
|   6| 19.74286|
|   8| 15.10000|
```
简单的几个命令就将，根据cyl列的值来计算mpg平均值的任务执行好，并且输出到屏幕中。

**实例四链接两个文件**

作者提到该功能还不是很完善，主要的缺陷有：

 - 用于连接命令后的第一个参数必须是现有文件，并且格式为（CSV或RDS）
 - 不能通过`by`连接指定参数，因此两个文件必须只有一个共同的列才能链接
 
这里我们会链接`cyl.csv`文件：

```
cat cyl.csv

####

#  cyl,description
#  4,four
#  6,six
```
通过`inner_join`命令来链接：

```
cat mtcars.csv | dplyr inner_join cyl.csv | dplyr kable

###结果
#  |  mpg| cyl|  disp|  hp| drat|    wt|  qsec| vs| am| gear| carb|description |
#  |----:|---:|-----:|---:|----:|-----:|-----:|--:|--:|----:|----:|:-----------|
#  | 21.0|   6| 160.0| 110| 3.90| 2.620| 16.46|  0|  1|    4|    4|six         |
#  | 21.0|   6| 160.0| 110| 3.90| 2.875| 17.02|  0|  1|    4|    4|six         |
#  | 22.8|   4| 108.0|  93| 3.85| 2.320| 18.61|  1|  1|    4|    1|four        |
#  | 21.4|   6| 258.0| 110| 3.08| 3.215| 19.44|  1|  0|    3|    1|six         |
#  | 18.1|   6| 225.0| 105| 2.76| 3.460| 20.22|  1|  0|    3|    1|six         |
#  | 24.4|   4| 146.7|  62| 3.69| 3.190| 20.00|  1|  0|    4|    2|four        |
#  | 22.8|   4| 140.8|  95| 3.92| 3.150| 22.90|  1|  0|    4|    2|four        |
#  | 19.2|   6| 167.6| 123| 3.92| 3.440| 18.30|  1|  0|    4|    4|six         |
#  | 17.8|   6| 167.6| 123| 3.92| 3.440| 18.90|  1|  0|    4|    4|six         |
#  | 32.4|   4|  78.7|  66| 4.08| 2.200| 19.47|  1|  1|    4|    1|four        |
#  | 30.4|   4|  75.7|  52| 4.93| 1.615| 18.52|  1|  1|    4|    2|four        |
#  | 33.9|   4|  71.1|  65| 4.22| 1.835| 19.90|  1|  1|    4|    1|four        |
#  | 21.5|   4| 120.1|  97| 3.70| 2.465| 20.01|  1|  0|    3|    1|four        |
#  | 27.3|   4|  79.0|  66| 4.08| 1.935| 18.90|  1|  1|    4|    1|four        |
#  | 26.0|   4| 120.3|  91| 4.43| 2.140| 16.70|  0|  1|    5|    2|four        |
#  | 30.4|   4|  95.1| 113| 3.77| 1.513| 16.90|  1|  1|    5|    2|four        |
#  | 19.7|   6| 145.0| 175| 3.62| 2.770| 15.50|  0|  1|    5|    6|six         |
#  | 21.4|   4| 121.0| 109| 4.11| 2.780| 18.60|  1|  1|    4|    2|four        |
```

介绍到这里就结束了，如果大家也喜欢这个实用的小工具，欢迎点击好看，转发该推文，支持我一波。

参考资料
https://github.com/coolbutuseless/dplyr-cli


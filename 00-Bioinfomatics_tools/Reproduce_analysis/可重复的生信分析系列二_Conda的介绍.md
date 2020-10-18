# 可重复的生信分析系列二:Conda的介绍

标签（空格分隔）： 可重复的生信分析

---

可重复的生信分析一直是未来的趋势。如果实现可重复的生信分析，关键在于分析软件版本的控制，一致的环境设置还有良好的分析流程的记录。Conda可以说是版本控制和生信工具安装的一大神器。相信大家对它了解肯定不少，但是又该怎么样利用它，进行可重复的分析呢？今天继续讲第二部分`Conda的介绍`。

本节教程将会使用到docker，去安装minconda的镜像。如果你还没看我docker的教程，强烈建议你先回顾一下:





## 什么是Conda?

### 基本介绍与其特点

从Conda的官方文件中找到了下面这一段解析:


> Conda是在Windows，macOS和Linux上运行的开源软件管理系统和环境管理系统。
> Conda可以快速安装，运行和更新软件包及其依赖的环境与工具。
> Conda可以轻松地在本地计算机上的环境中创建，保存，加载和切换。它是为Python程序创建的，但可以适用于任何语言的软件。

如果你在安装生物信息学工具之前遇到过依赖性问题，Conda会轻而易举帮助您解决这一问题。此外，Conda使安装和使用不兼容的工具变更加容易。因为它可以创建不同的虚拟环境，使得不兼容的工具在相对独立的环境中运行，两者之间不打架。

### 关于几个conda

**什么是Anaconda?**

Anaconda是Conda的发行的一个安装包。它是一个数据科学平台，其中包含许多软件包（有点太多了个人认为）。

**什么是Miniconda?**

就如其名，Miniconda是Conda的最小安装程序。可以看作是小型版本的Anaconda，仅包含Conda，Python，它们依赖的软件包以及少量其他有用的软件包，包括pip，zlib和其他一些软件包。对于生信分析而言，个人更喜欢使用Miniconda并仅安装所需的工具。

**什么是Bioconda?**

Bioconda是conda生物信息学软件的托管平台。Conda通道只是存储软件包的位置。但Bioconda管道上提供了使用最广泛的生物信息工具，该工具托管了超过6,000多种生物信息包。

### 使用docker安装,使用conda

将上节课所学的知识运用，使用docker来安装conda:

```
docker pull continuumio/miniconda3
```

使用docker，运行新的conda container:

```
# run new container
docker run -it --rm continuumio/miniconda bash
```

运行上面的命令后，可以看到类似于这样的terminal：

```
(base) root@c94ca01d4fdb:/#
```

像其它工具一样，直接输入conda并且不带任何参数，就可以得到其帮助文档:


```
conda

usage: conda [-h] [-V] command ...

conda is a tool for managing and deploying applications, environments and packages.

Options:

positional arguments:
  command
    clean        Remove unused packages and caches.
    config       Modify configuration values in .condarc. This is modeled
                 after the git config command. Writes to the user .condarc
                 file (/root/.condarc) by default.
    create       Create a new conda environment from a list of specified
                 packages.
    help         Displays a list of available conda commands and their help
                 strings.
    info         Display information about current conda install.
    init         Initialize conda for shell interaction. [Experimental]
    install      Installs a list of packages into a specified conda
                 environment.
    list         List linked packages in a conda environment.
    package      Low-level conda package utility. (EXPERIMENTAL)
    remove       Remove a list of packages from a specified conda environment.
    uninstall    Alias for conda remove.
    run          Run an executable in a conda environment. [Experimental]
    search       Search for packages and display associated information. The
                 input is a MatchSpec, a query language for conda packages.
                 See examples below.
    update       Updates conda packages to the latest compatible version.
    upgrade      Alias for conda update.

optional arguments:
  -h, --help     Show this help message and exit.
  -V, --version  Show the conda version number and exit.

conda commands available from other packages:
  env
```

### conda的版本

在安装工具前，先确保我们使用的conda版本是最新的:

```
### 查看当前版本
conda --version
conda 4.8.2

###进行升级，会弹出需要你输入y的确认信息
conda update conda

###再次查看更新后的版本
conda --version
conda 4.8.3

###另外，你还可以将所有conda的包更新为最新兼容的版本(可选的)
conda update --all
```

对于conda下载的缓存临时文件，可以定期通过`conda clean`进行清理:

```
conda clean -a
```

### 使用conda安装生物信息软件bcftools

在没有conda之前，我们需要 通过一系列的操作，才能完成安装:

![][1]

相比之下，使用Bioconda的管道进行安装就容易得多了:

```
### 使用Bioconda的管道进行安装
conda install -c bioconda bcftools
###下载测试文件
cd /tmp
wget https://github.com/davetang/learning_vcf_file/raw/master/aln_consensus.bcf

# 查看前面两个的SNP信息
bcftools view -v snps aln_consensus.bcf | grep -v "^#" | head -2
1000000 336 .   A   G   221.999 .   DP=112;VDB=0.756462;SGB=-0.693147;MQ0F=0;AF1=1;AC1=2;DP4=0,0,102,0;MQ=60;FQ=-281.989    GT:PL   1/1:255,255,0
1000000 378 .   T   C   221.999 .   DP=101;VDB=0.704379;SGB=-0.693147;MQ0F=0;AF1=1;AC1=2;DP4=0,0,99,0;MQ=60;FQ=-281.989 GT:PL   1/1:255,255,0

```

## Conda的环境管理

相信大部分的小伙伴对上面提到的分析都应该了如指掌了，但是conda在可重复的生信分析中，究竟能起到一个什么的作用，下面请听我细说：

### 什么是Conda的环境? 其环境有什么用？

使用Conda，你可以为某个项目或者某个分析创建一个独特隔离的环境。换个意思，所谓的环境就是一组可在一个或多个项目中使用的软件包。Miniconda的默认环境是base环境。

我强烈不建议在同一环境中安装所有软件包/工具。这个是很多**新手玩家**会犯的错误。很多刚刚入门生信的初学者，都会一个劲的在base的环境里，安装各种各样他们所需的工具。因为贪图简便，一个`conda install`就觉得自己可以走天下了。但是随着软件越装越多，因为不同的软件所需的依赖包不同，就会造成当你安装某系软件后，你之前安装的一些软件就无法运行了。举个例子，比如有些软件是依赖python3的，但是有一些是依赖python2，当你安装新的软件时，conda会自动帮你把当前的环境转换成你该工具所依赖的环境。这样一来，就相当于拆东墙补西墙，很多软件都互相打架了，然后实在没办法了，嗯，就把conda删了，重新把所有软件安装一遍。不要问我为什么那么清楚，因为我也踩过坑。

目前有两种创建conda环境的方法:

 1. 通过环境文件YAML来创建(`environment.yml`)
 2. 通过命令来手动指定需要安装的软件包

### 通过环境文件来创建conda环境

首先看看一个`environment.yml`文件的例子:

```
name: bwa_old
channels:
  - bioconda
dependencies:
  - bwa=0.7.15
```

该文件主要由三部分组成:

 - 该环境的名字`name:`
 - 你所指定的channel去安装工具`channels`
 - 所需安装的软件包

现在我们开始构建我们的环境:

```
#下载好预先准备好的环境文件
wget https://raw.githubusercontent.com/davetang/reproducible_bioinformatics/master/environment.yml
#通过environment.yml文件构建环境
conda env create --file environment.yml

# 检查当前conda所有的环境
conda env list
##可以看到当前的环境有两个
# conda environments:
#
base                  *  /opt/conda
bwa_old                  /opt/conda/envs/bwa_old
```

 安装好环境后，要怎么使用呢？首先需要激活已经安装好的环境:
 
```
 conda activate bwa_old

# 你的terminal就会变成提示你你已经切换成(bwa_old)这个环境了
# (bwa_old) root@d470a3e9da91:/tmp#

### 查看一下bwa，看看是不是确实安装好了
bwa

Program: bwa (alignment via Burrows-Wheeler transformation)
Version: 0.7.15-r1140
Contact: Heng Li <lh3@sanger.ac.uk>

``` 
如果你运行完bwa后，想切回到之前的原始环境可以使用`conda deactivate`:

```
conda deactivate

# 回到之前的base了
# (base) root@d470a3e9da91:/tmp#
```

### 通过命令来手动指定需要安装的软件包

除了通过指定的环境文件来构建conda的环境之外，我们还可以通过手动指定需要安装的软件包来构建我们所需的环境。

下面看一个例子:

```
conda create -c conda-forge -n test_env python=2.7 numpy matplotlib pandas
```

这里我就手动告诉conda，通过`-c`说明安装的channel是conda-forge,通过`-n`告诉它我所创建的环境的名字,最后所需的安装包跟在最后面`python=2.7 numpy matplotlib pandas`。这样conda就会自动帮你处理好不同软件包之间的依赖项，完成安装。

对于两种安装方式而言，第一种是我个人更喜欢的形式。为什么呢？

### 与它人分享你的环境

通过`environment.yml`文件，我们可以轻松的将你分析所用的文件分享给别人。这一样一来，能确保所用的软件版本，分析的环境都是一致的。通过保存的不同的YML环境文件，我们可以清楚，方便的管理conda的每一个环境。

下面给大家一个例子，如果分享我们刚刚构建的`bwa_old`环境：

```
##激活`bwa_old`环境
conda activate bwa_old
### 输出当前环境的文件配置
conda env export -f test_env.yml
### 查看一下当前环境的文件配置
cat test_env.yml 
name: bwa_old
channels:
  - bioconda
  - defaults
dependencies:
  - _libgcc_mutex=0.1=main
  - bwa=0.7.15=1
  - libgcc=7.2.0=h69d50b8_2
  - libgcc-ng=9.1.0=hdf63c60_0
  - libstdcxx-ng=9.1.0=hdf63c60_0
  - zlib=1.2.11=h7b6447c_3
prefix: /opt/conda/envs/bwa_old
```
这时候，我们只要简单的将`test_env.yml`发给我们的合作者，这样他们就可以使用到和咱们一样的环境啦。

### 关于conda的一些杂七杂八

主要的内容介绍得差不多了，下面简单总结一下关于conda剩下的一些杂七杂八。

如何删除一个环境：

```
conda env remove -n bwa_old
```

查看当前环境下所安装的包:

```
conda list -n bwa_old
# packages in environment at /opt/conda/envs/bwa_old:
#
# Name                    Version                   Build  Channel
_libgcc_mutex             0.1                        main  
bwa                       0.7.15                        1    bioconda
libgcc                    7.2.0                h69d50b8_2  
libgcc-ng                 9.1.0                hdf63c60_0  
libstdcxx-ng              9.1.0                hdf63c60_0  
zlib                      1.2.11               h7b6447c_3  
```

指定你喜欢的安装渠道,预先配置这些通道，以便每次创建环境时,都不需要指定下载的渠道:

```
conda config --add channels conda-forge
conda config --add channels bioconda
```

删除不想要的conda包:

```
conda remove bcftools
```

关于conda的介绍就到这里了，希望你有所收获。关于conda的使用，还有很多的细节还没完全介绍完。你是否在conda使用时遇到过坑，欢迎在留言区留言，和大家分享你曾踩过的坑。

参考整理链接:
https://davetang.github.io/reproducible_bioinformatics/conda.html



 
  [1]: http://static.zybuluo.com/lakesea/y9vpicinl2rw33bp0hkrngel/bcftools.png
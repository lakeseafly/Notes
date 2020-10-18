# 比对文件BAM/CAM深度检测好用工具：Mosdepth

标签（空格分隔）： 生信工具

---

做完比对后，最直接的一个问题就是想知道：你的reads比对上了哪些区域没比对上哪些区域，有多少reads比对上特定的区域(该区域比对深度)。传统手艺一般可以使用`bedtools cov`或者`samtools depth`来直接实现，但是受限其速度，这些工具我个人用起来体验不太好，特别是对文件大小很大的BAM file。今天就给大家推荐一款我觉得非常好用，速度很快的比对文件BAM/CAM深度检测好用工具，**Mosdepth**。

### 工具简介

Mosdepth是一种新的命令行工具，用于快速计算全基因组测序覆盖率。它测量BAM或CRAM文件在基因组中每个核苷酸位置或基因组区域的深度。可以将基因组区域指定为被BED文件覆盖的指定区域，或者指定为CNV calling所需的固定大小窗口。Mosdepth使用一种计算效率高的简单算法，使其能够快速生成覆盖摘要。Mosdepth比现有工具更快，并且提供了所产生的覆盖剖面类型的灵活性。


### 工具特点

总的来说，Mosdepth有以下几项比较显著的特点:

 - 对比bedtools, samtools 速度快
 - 可以根据给定窗口大小来计算平均每个窗口深度，用于CNV calling
 - 能根据BED文件给出其覆盖的指定区域的深度
 - 量化输出，它合并相邻的碱基，只要它们都同时落入相同的覆盖深度的区域，例如（10X-20X)。
 - 能给出每个染色体和全基因组在特定阈值或以上覆盖的碱基比例分布。
 - 能够设定阀值输出，用于表示在给定阈值处覆盖每个区域中有多少个碱基。
 - 能给出每条染色体和每条染色体特定区域内平均深度的总结。

### 工作原理

当遇到每条染色体时，mosdepth会在染色体的长度上创建一个数组。对于遇到的每个起始位置，它会增加数组位置的值。对于每个停止的位置，它会减少该位置。由此，特定位置处的深度是其前面的所有阵列位置的累积和（在BEDTools中使用类似的算法，其中单独跟踪开始和停止）。 mosdepth避免重复计算重叠的配偶对，并使用CIGAR操作跟踪每个读取的每个比对上的部分。由于这种数据结构，可以在不显着增加运行时间的情况下完成覆盖分布计算。下图显示了这个概念：

![][1]

这个数组计算非常快。因为其没有额外的分配或跟踪对象，它在概念上也很简单。由于这些原因，它比`samtools depth`更快。

### 工具安装
目前最新版是v 0.2.6,安装方法有很多可以直接下载编译好的版本:

```
###下载地址
https://github.com/brentp/mosdepth/releases
```

或者直接通过conda来安装

```
conda install mosdepth
```


### 使用说明
 
```
mosdepth 0.2.3

  Usage: mosdepth [options] <prefix> <BAM-or-CRAM>

Common Options:

  -t --threads <threads>     number of BAM decompression threads. (use 4 or fewer) [default: 0]
  -c --chrom <chrom>         chromosome to restrict depth calculation.
  -b --by <bed|window>       optional BED file or (integer) window-sizes.
  -n --no-per-base           dont output per-base depth. skipping this output will speed execution
                             substantially. prefer quantized or thresholded values if possible.
  -f --fasta <fasta>         fasta file for use with CRAM files.

Other options:

  -F --flag <FLAG>              exclude reads with any of the bits in FLAG set [default: 1796]
  -i --include-flag <FLAG>      only include reads with any of the bits in FLAG set. default is unset. [default: 0]
  -x --fast-mode                dont look at internal cigar operations or correct mate overlaps (recommended for most use-cases).
  -q --quantize <segments>      write quantized output see docs for description.
  -Q --mapq <mapq>              mapping quality threshold [default: 0]
  -T --thresholds <thresholds>  for each interval in --by, write number of bases covered by at
                                least threshold bases. Specify multiple integer values separated
                                by ','.
  -R --read-groups <string>     only calculate depth for these comma-separated read groups IDs.
  -h --help                     show help
```
 
 简单说说我平时常用到的参数：
 

 - `-t`使用线程的数目，这里推荐少于4个CPU，因为文档有说到多于4个线程，速度不会有提升
 - `-c`指定具体哪个染色体去计算深度
 - `-b`能根据BED文件给出其覆盖的指定区域的深度,比如你给出的BED文件含有一个物种对应基因的位置，就能计算其基因的覆盖度。
 - `-F`在计算覆盖度时，排除BAM中含有该FLAG的reads。默认是1796，代表着:read unmapped (0x4), not primary alignment (0x100)， read fails platform/vendor quality checks (0x200)，  read is PCR or optical duplicate (0x400)。一般设定为默认就好了。
 - `-Q` 比对最低质量的要求，默认是0，一般可以按照自己的需要进行调整为5或者10。
 - `-x` 快速模式，推荐给大型的BAM文件使用
 - `-T` 按照阀值（比如1X,2X,10X）输出达到指定阀值的核苷酸数目。


 这里可以根据自己的需要使用对应的参数。下面给出一些常用的命令行运行例子以供大家参考使用:
 
根据bed文件来计算指定区间的覆盖度:
```
mosdepth -b capture.bed sample-output sample.exome.bam
```
 进一步使用`-T`根据指定阀值进行计算:
 
```
mosdepth --by exons.bed --thresholds 1,10,20,30 $prefix $bam
```
 
 输出效果如下:
 
```
#chrom  start   end     region           1X   10X  20X  30X
1       11869   12227   ENSE00002234944  358  157  110  0
1       11874   12227   ENSE00002269724  353  127  10   0
1       12010   12057   ENSE00001948541  47   8    0    0
1       12613   12721   ENSE00003582793  108  0    0    0
```


该工具我个人体验非常好，又快又好用，输出结果简单易懂。如果有需要计算比对覆盖需求的小伙伴可以试一试哦。最后给大家附上其github链接以便大家进一步研究使用该工具:https://github.com/brentp/mosdepth



  [1]: http://static.zybuluo.com/lakesea/rvyfv0stwii4lo0kfbxgrtmz/29647913-d79ab028-8848-11e7-86cf-60d4b087bc3b.png
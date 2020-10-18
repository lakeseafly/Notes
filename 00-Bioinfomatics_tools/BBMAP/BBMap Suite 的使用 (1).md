# 生信小工具专题：BBTools/BBMap Suite 的使用 (1)

标签（空格分隔）： 生信工具

---

> 不会编程的小白通常会烦恼怎样去处理一些简单的数据统计，不要紧其实日常我们用到百分之80以上的数据统计分析，都已经有人帮我们将轮子造好了，只要我们学会怎样利用好这些工具，很多事情都可以事半功倍。这周开始会给大家介绍一些常用的生信工具，并且提供一些简单的测试数据供大家学习玩耍。第一篇文章首先讲BBMap。


###简单介绍


BBMap/BBTools是一种用于DNA和RNA测序reads的拼接感知全局的比对工具。它可以处理的reads包括，Illumina，454，Sanger，Ion Torrent，Pac Bio和Nanopore。 BBMap快速且极其准确，特别是对于高度突变的基因组或长插入读数，甚至超过100kbp长的全基因缺失。它对基因组大小或contigs数量没有上限。并且已经成功地用于绘制到具有超过2亿个contigs的85gb的土壤宏基因组。此外，与其他比对工具相比，索引阶段非常快。
BBMap有很多选项，在shell脚本中描述。它可以输出许多不同的统计文件，例如经验读取质量直方图，插入大小分布和基因组覆盖，有或没有生成sam文件。因此，它可用于文库的质量控制和测序运行，或评估新的测序平台。衍生程序BBSplit也可用于分类或精炼宏基因读取。

那问题来了，现在的工具那么多为什么会推荐还有写这个bbmap、bbTools的教程？那当然是因为该工具有一些它自身的特点：

 1. BBTools是用纯Java编写的，可以在任何平台（PC，Mac，UNIX）上运行，除了安装Java之外没有依赖项（为Java 6及更高版本编译）。所有工具都是高效且多线程的。
 2. BBTools可以处理常见的排序文件格式，如fastq，fasta，sam，scarf，fasta + qual，压缩或raw，具有自动检测质量编码和交错。
 3. BBTool套件包括对高通量测序数据进行质量控制，修剪(trim)，纠错，组装和比对的程序。
 4. BBTools是开源的，可以无限制免费使用，不用钱当然好！
 5. BBTools无需安装。从BBTools SF站点下载tar存档后，您需要做的就是解压，很动心是不是？

###安装与注意事项

经过简单的介绍后，大家相信对这一款工具也有一定的了解，下面开始安装该工具。这里的安装介绍都是基于Linux系统，Windows系统的运行方式有点不同，在这里没有介绍。

conda 大法好，简单一个command帮你搞定安装,如果你还不知道什么是conda，可以先阅读一下大神的博客 （http://www.bio-info-trainee.com/1906.html），如果你还是没有弄懂，或者你想了解更多关于文件安装之类的话题，那么我强力推荐你去观看由我们大号生信技能树所推出的一个课程：

```
conda install -y bbmap

```

当然如果你不会使用conda，也没问题，直接去到bbmap的网站下载解压就好

```
##下载
wget https://excellmedia.dl.sourceforge.net/project/bbmap/BBMap_38.16.tar.gz
##解压
tar -xvfz BBMap_38.16.tar.gz
```


注意：BBMap（和所有相关工具）shellcripts将尝试自动检测内存，但可能会失败（导致`jvm`无法启动或内存不足），具体取决于系统配置。这可以通过将`-XmxNNg`标志添加到shellscript的参数列表（根据计算机的物理内存进行调整，不要使用超过总RAM的80％）来覆盖，并将其传递给java。

BBTools程序接受经过gzip压缩的fastq文件，并能够以多种数据格式写入输出。同时支持以fq 或者 fastq结尾的测序文件。

###牛试小刀
在按照工具类别对BBmap/BBtools进行逐步介绍时，先介绍几款其在统计数据中简单但有好用的小工具。

####stats (stats.sh)

用于统计reads或者assembly基本的信息，例如 scaffold count, N50, L50, GC content, gap percent。 

```
#用一个fastq文件作为例子，这个工具也可以用于fasta文件
stats.sh in=A1.1.fastq

#结果
Main genome scaffold total:             250000
Main genome contig total:               250000
Main genome scaffold sequence total:    22.500 MB
Main genome contig sequence total:      22.500 MB       0.000% gap
Main genome scaffold N/L50:             250000/90
Main genome contig N/L50:               249990/90
Main genome scaffold N/L90:             250000/90
Main genome contig N/L90:               249990/90
Max scaffold length:                    90
Max contig length:                      90
Number of scaffolds > 50 KB:            0
% main genome in scaffolds > 50 KB:     0.00%


Minimum         Number          Number          Total           Total           Scaffold
Scaffold        of              of              Scaffold        Contig          Contig
Length          Scaffolds       Contigs         Length          Length          Coverage
--------        --------------  --------------  --------------  --------------  --------
    All                250,000         250,000      22,500,000      22,499,990   100.00%
     50                250,000         250,000      22,500,000      22,499,990   100.00%
```

如果你有多个输入文件，可以使用`statswrapper.sh`工具,对多个数据同时进行运算：

```
statswrapper.sh *.fastq

n_scaffolds     n_contigs       scaf_bp contig_bp       gap_pct scaf_N50        scaf_L50        ctg_N50 ctg_L50 scaf_N90        scaf_L90        ctg_N90 ctg_L90 scaf_max        ctg_max scaf_n_gt50K    scaf_pct_gt50K  gc_avg  gc_std filename
250000  250000  22500000        22499990        0.000   250000  90      249990  90      250000  90      249990  90      90      90      0       0.000   0.48428 0.05525 /scratch/pawsey0149/hhu/bio_trainee/tool_bbmap/data/A1.1.fastq
250000  250000  22500000        22499464        0.002   250000  90      249703  90      250000  90      249703  90      90      90      0       0.000   0.48673 0.05146 /scratch/pawsey0149/hhu/bio_trainee/tool_bbmap/data/A1.2.fastq
```
该工具在统计多个fastq文件的基础信息时非常方便。

####pileup (pileup.sh)

用于计算scaffold的覆盖率，需要没有sort过的bam 或者 sam 的格式的文件作为input。

```
 pileup.sh in=test.sam out=test.out
 
 #屏幕输出 一些基本的比对信息，还有比对使用的ref的信息
 
Reads:                                  66234
Mapped reads:                           57247
Mapped bases:                           8587050
Ref scaffolds:                          9
Ref bases:                              41030279

Percent mapped:                         86.431
Percent proper pairs:                   0.000
Average coverage:                       0.209
Standard deviation:                     0.583
Percent scaffolds with any coverage:    100.00
Percent of reference bases covered:     15.45

#test.out里面的输出，每一个染色体上比对的覆盖率等信息。
#ID     Avg_fold        Length  Ref_GC  Covered_percent Covered_bases   Plus_reads      Minus_reads     Read_GC Median_fold     Std_Dev
1       0.2132  7978604 0.0000  15.6789 1250959 5765    5577    0.5149  0       0.59
2       0.2127  8319966 0.0000  15.6720 1303906 5940    5860    0.5195  0       0.59
3       0.2031  6606598 0.0000  15.0522 994440  4417    4531    0.5205  0       0.58
4       0.2047  5546968 0.0000  14.9938 831704  3795    3775    0.5218  0       0.58
5       0.2126  4490059 0.0000  15.6697 703581  3183    3182    0.5188  0       0.59
6       0.2124  4133993 0.0000  15.8143 653760  2973    2882    0.5202  0       0.58
7       0.2068  3415785 0.0000  15.3674 524917  2315    2394    0.5139  0       0.58
supercont8.8    0.1830  535760  0.0000  14.1227 75664   325     329     0.5165  0       0.52
mit     0.2357  2546    0.0000  23.5664 600     2       2       0.6050  0       0.42


```

###readlength (readlength.sh)

统计read的平均长度，总长度等数据信息


```
readlength.sh in=A1.1.fastq in2=A1.2.fastq

#Reads: 500000
#Bases: 45000000
#Max:   90
#Min:   90
#Avg:   90.0
#Median:        90
#Mode:  90
#Std_Dev:       0.0
#Read Length Histogram:
#Length reads   pct_reads       cum_reads       cum_pct_reads   bases   pct_bases       cum_bases       cum_pct_bases
90      500000  100.000%        500000  100.000%        45000000        100.000%        45000000        100.000%
```

###kmercountexact (kmercountexact.sh)

计算文件中unique kmers的数量。生成kmer频率直方图和基因组大小估计。

```
kmercountexact.sh in=A1.1.fastq in2=A1.2.fastq out=kmer_test.out khist=kmer.khist peaks=kmer.peak

##屏幕输出
Input:                          500000 reads            45000000 bases.

For K=31
Unique Kmers:                   3326507
Average Kmer Count:             9.016
Estimated Kmer Depth:           9.015
Estimated Read Depth:           13.526

##新生成三个文件
kmer_test.out ##运行输出，所以kmer的fa集合
kmer.khist ##成kmer频率直方图列表
kmer.peak ##kmer统计size等峰值的统计信息


```


###bbmask (bbmask.sh)

Masks低复杂度或包含重复kmers的序列，或由比对到的reads所覆盖的序列。默认情况下的参数：a window=80 and entropy=0.75

输入可以是stdin或fasta或fastq文件，sam也是可选的文件格式。

```
### 其中一个例子
bbmask.sh in=A1.1.fastq out=test.mark

### 屏幕输出

Masking low-entropy (to disable, set 'mle=f')
Low Complexity Masking Time:  	0.479 seconds.
Ref Bases:                  22500000 	46.97m bases/sec
Low Complexity Bases:             88

Converting masked bases to N
Done Masking
###这里就是将fastq文件进行简单重复的屏蔽，将其标记为N，这里可以看到有88个被标记为N
```


----------
今天就简单介绍到这，有条件的同学可以用我提供的测试数据进行理解与学习。下一个推文会按照BBmap工具功能进行分类并详细介绍相应的工具功能。


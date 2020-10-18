# 生信工具：ppsPCP-植物存在/缺失变异扫描和泛基因组构建工具

标签（空格分隔）： 生信小工具

---

###工具简介

在原核生物的泛基因组研究中早早已经有很多工具被设计并使用。对于植物来说，由于其基因组比较复杂，比较大，一直没有很完善的流程被设计出来。该推文主要和大家介绍植物泛基因组流程ppsPCP。它能够扫描存在/缺失变异（PAV）并构建完全注释的泛基因组。该工具将有助于植物泛基因组研究，并帮助我们研究基因的存在、缺失变异与遗传/表型变异和基因组多样性的联系。当然以我个人的见解是，该工具也适合其它物种关于存在/缺失变异和构建泛基因组的研究。



###背景简介

为了预防大家对这个概念或者领域比较陌生，下面会简单的和大家介绍一下其背景知识。

2005年，Tettelin等人提出了微生物泛基因组概念（pangenome，pan源自希腊语‘παν’，全部的意思），泛基因组即某一物种**全部基因的总称**。2009 年，Li等人首次采用新全基因组组装方法对多个人类个体基因组进行拼接，发现了个体独有的DNA序列和功能基因，并首次提出了“人类泛基因组”的概念，即人类群体基因序列的总和。2009 年泛基因组测序首次应用于人类基因组学研究；2013年泛基因组测序应用于动植物研究领域;2019年第一个泛基因组研究在大规模的(910名)非洲人后裔中开展。源源不断中，还有很多很多泛基因组的研究正在进行中。可以看出，随着测序技术的进步和其价格的下降，泛基因组研究变得越来越热门。

![][1]


如图，泛基因组进而可以分为，**核心基因组**（core genome）和**可变基因组** (variable genome)。核心基因指的是，在所有动植物品系或者菌株中都存在的基。可变基因组是指，在1个以及1个以上的动植物品系或者菌株中存在的基因。如果某个基因，仅存在某一个动植物品系或者菌株中，该基因还可以细分为品系或者菌株特有基因。一般来说，核心基因组控制着生命体基本生成代谢的功能。另外，结构变异中的**存在/缺失变化**(presnece/absence variation)是泛基因组的重点研究对象，因为可变基因组可能就是使个体产生不同性状（抗病性，抗寒性等）的原因。


###泛基因组的构建法：重头组装构建法



这是构建泛基因组最经典的方法，最早运用于构建细菌的泛基因组。分别对多个个体进行，分别的重头组装和注释，然后将所得的每个个体的新组装的序列与参考序列reference基因组进行比对，找出比对不上的区域，进行整合。对于比较大的基因组，此方法需要耗费更多的电脑资源，因为需要对每一个个体进行分别进行重头组装。那为什么要提这个构建方法呢，因为即将介绍的工具是基于这种泛基因组的构建方法。


###工具的安装

介绍了那么多，差不多可以开始接触这个工具了。首先是要把这个工具还有其需要依赖的工具一一安装好。

**ppsPCP的安装**
简单地从git hub上拷贝下来就好了
```
git clone git@github.com:Zhuxitong/ppsPCP.git
export PATH=/path/to/ppsPCP/bin/:$PATH
```
**依赖包**

**MUMmer**
这里一定要下载并使用 Mummer-4.0.0beta2的版本，如果使用其它版本，根据我的测试是会报错的。估计这个工具写的时候，是按照只适用于该版本的Mummer进行编写的。
```
$ wget https://github.com/mummer4/mummer/releases/download/v4.0.0beta2/mummer-4.0.0beta2.tar.gz
$ tar -xvzf mummer-4.0.0beta2.tar.gz
$ ./configure --prefix=/path/to/installation
$ make
$ make install
# Add MUMmer tools to your PATH
$ export PATH=/path/to/installation/:$PATH
```

**Blast+**

经我测试对版本没有绝对的要求，如果你系统已经按照好了旧的blast+，依然是可以运行的。当然不推荐用很旧很旧的版本（好几年没更新过那种）。
```
$ wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-2.9.0+-x64-linux.tar.gz
$ tar zxvf ncbi-blast-2.9.0+-x64-linux.tar.gz
# Add Blast+ tools to your PATH
$ export PATH=/path/to/blast+/bin:$PATH
```


**Bedtools**
这里我要说，这个bedtools一定要使用最新的版本，一开始我根据该工具的manual使用bedtoolsv v2.5 结果就是无论怎样，运行到最后一步bedtools要index fasta这一步都会出错。
```
$ wget https://github.com/arq5x/bedtools2/releases/download/v2.28.0/bedtools-2.28.0.tar.gz
$ tar -zxvf bedtools-2.28.0.tar.gz
$ cd bedtools2
$ make
$ export PATH=/path/to/bedtools/bin:$PATH
```

**Blat和gffread**

这几个就正常跟着流程按照就好了，对版本要求不大。
```
###blat
$ mkdir UCSC_tools
$ rsync -aP rsync://hgdownload.soe.ucsc.edu/genome/admin/exe/linux.x86_64/ ./
# Add blat to your PATH
export PATH=/path/to/UCSC_tools/blat/:$PATH
###gffread 它是cufflinks中的一个小工具，所以通过按照cufflinks就能调用了
$ wget http://cole-trapnell-lab.github.io/cufflinks/assets/downloads/cufflinks-2.2.1.Linux_x86_64.tar.gz
$ tar zxvf cufflinks-2.2.1.Linux_x86_64.tar.gz
# Add gffread to your PATH
$ export PATH=/path/to/cufflinks-2.2.1.Linux_x86_64/:$PATH

```

**Perl module**
最后就是安装bioperl这个模块，因为这个工具其中的几个脚本都有调用到这个模块

```
$ cpanm --local-lib=~/perl5 local::lib && eval $(perl -I ~/perl5/lib/perl5/ -Mlocal::lib)
$ cpanm Bio::Perl
```



###工具使用说明


依赖的包不少，但使用的方法却很简单，只要将参考基因组和参考基因组的注释，再加上其它不同的植株所对应的组装和注释放到命令中就行了。换句话讲，这就要求你事先已经组装好每一个不同植株或者个体，并且将其注释好（为了确保准确性和一致性这里一般都会使用相同的工具进行组装和注释）。
```
Usage: 
make_pan.pl [options] --ref [reference_genome] --ref_anno [refernece_anno] --query query1_genome[query2...] --query_anno query1_anno[query2...] 
```

其它比较重要的参数就是`--coverage`，`--sim_gene`和`--sim_pav`，基因存在/缺失的变异，将由序列的深度，相似度来决定。然后就是`--thread `,使用更多的CPU能够在mummer和blast的步骤极大提高运行的效率。

```
***Filter parameters
--coverage      The coverage used to filter similar PAVs. Can be any number between 0 and 1. Default: 0.9
--sim_pav       The similarity used to filter similar PAVs. Can be any number between 0 and 1. Default: 0.95
--sim_gene      Then similarity used to filter mapped genes in blat mapping. Can be any number between 0 and 1. Default: 0.8

--thread        The number of threads used for mummer and blastn. Remember not all the phases of ppsPCP are parallelized. Default: 1
```



###流程概述

![][2]


接着就让我们更加细致了解一下这个工具所涉及的流程，主要步骤分为十步:

 1. 全基因组de novo assembly，然后每个材料的全基因组和参考基因组比对去找出新的独特的non-references的序列。这一步会使用到MUMmer。
 2. 比对的结果回用于扫描PAVs，最短的PAV长度是被定义为100bp。
 3. 为了确认这些PAV，BLASTn用于在参考基因组上搜索这些PAV序列，确认他们是存在/缺失。
 4. 将上面BLASTn的结果用来判断PAVs序列可不可信，分别有两种情况:1)与参考序列非常接近(相似度>=95% 和该片段>=90%的序列被覆盖到)，这种情况下,这些高度相似的片段会被过滤掉。2)PAVs序列在参考基因组中找不到。另一句话说他们是缺失的。要被选出来。
 5. 这些被选出来的PAVs，进一步和query片段比较，如果他们是重叠的，就进而延长他们。
 6. 所有被挑选出来并延长的PAVS片段，会挪到一起，每个片段之间使用100bp的N-bases相连.然后准备进行注释
 7. 最后这些新的PAVS片段和参考基因组就组成了泛基因组
 8. 每个材料的基因组和参考基因组比对使用BLAT工具。
 9. 提取的基因区域是从query基因组和新生成二代PAV基因组中挖掘出来的（第五步中的结果）
 10. 一个完整的基于参考基因组的泛基因组，是通过合并PAVS序列文件及其与参考基因组的注释来构成泛基因组的

###测试运行

使用其提供的测试文件进行测试

```
make_pan.pl --ref Zmw_sc00394.1.fa --ref_anno Zmw_sc00394.1.gff3 --query Zjn_sc00188.1.fa --query_anno Zjn_sc00188.1.gff3
```

生成了两个文件:`pangenome1.fa` 和`pangenome1.gff3`。简单解析一下，`pangenome1.fa`就是该测试数据所生成的泛基因组，其由参考基因组`Zmw_sc00394.1.fa`和另一个isolate`Zjn_sc00188.1.fa`组装中和参考基因组不相同的序列组成(相当于non-refernce sequences，Zjn_sc00188.1 isolate中独有的基因)。`pangenome1.gff3`就是由参考基因中的基因注释加上只存在于Zjn_sc00188.1中的PAVS 基因(存在/缺失)注释共同组成。

介绍到这里就差不多结束了，除了该测试数据外，我个人也很有兴趣去使用更大的测试数据例如rice的去测试该方法找到的genes PAV和 mapping based 的方法找到的genes PAV的差异。


该工具包链接：
https://github.com/Zhuxitong/ppsPCP





  [1]: https://static.xmt.cn/3835159655856091237.png
  [2]: http://cbi.hzau.edu.cn/ppsPCP/images/pipeline.png
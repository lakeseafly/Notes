# BLAST本地比对太慢？不怕用diamond

标签（空格分隔）： 生信工具

---

在宏基因组学研究中，又或者你准备做大规模的系统进化的相关研究等等，需要分析了数百万个序列的reads，以确定来自环境的微生物样品的功能或分类学含量，找出对应reads同源的物种。这里重要的步骤是确定存在哪些基因，通常通过将翻译的DNA序列与蛋白质序列的参考数据库比对，例如NCBI非冗余（NCBI-nr）蛋白质数据库。长期以来，BLASTX因其高灵敏度而被认为是此标准的黄金标准工具。但是，BLASTX对于处理大批量的数据来说还是太慢了（就算你将你的蛋白质切成很小份并行分析，也需要一段长时间的运行，并且会耗费大量的计算机资源)。

今天的推文给大家介绍一下钻石(DIAMOND)，目前的引用量已经超过1000，一款非常适合在处理大数据量的蛋白质或者核苷酸序列分析中替换BLASTX/BLASTP。据分析，当针对NCBI-nr数据库进行显着比对，预期值低于10 -3时，DIAMOND比BLAST比对大约快20,000倍于，并具两个工具有相似的灵敏度水平。



###软件基本介绍

DIAMOND是一种高通量比对程序，可将DNA测序reads文件与蛋白质参考序列文件（如NCBI-nr）进行比较。它以C ++实现，旨在在多核服务器上快速运行。

![][2]


该程序明确地设计为，利用具有大内存容量和许多内核的现代计算机体系结构。那么为什么它那么快呢，因为它使用了种子和延伸方法。额外的算法成分是使用缩小的字母，间隔种子和双索引。算法简单了解一下就可以了，具体的算法的内容比较难懂就不深入讨论了。


其主要的特点包括(为什么该工具那么受欢迎):

 - 在比较NCBI-nr数据库的短DNA reads时，DIAMOND比BLASTX 快4个数量级，并且在e-value < 0.001 的比对上保持相当的灵敏度水平。简单一句话就是又快又准。
 - 可用于移位分析的长reads比对
 - 资源要求低，适合在标准台式机或笔记本电脑上运行(入门门槛低，适合各类的玩家)
 - 各种输出格式，包括BLAST对比格式，例如格式6的tabular分隔形式和格式5的XML格式 (很多下游分析，或者脚本都是基于BLAST的输出结果，diamond能够直接输出和Blast相同的格式不能不说是最大的优点之一)


###Dimaond的安装和使用

Diamond的安装非常简单，因为作者已经把预先编译好的版本准备好了，只需下载和解压，将diamond路径放入环境中就能够方便使用了:

```
wget http://github.com/bbuchfink/diamond/releases/download/v0.9.24/diamond-linux64.tar.gz
tar xzf diamond-linux64.tar.gz
```

和BLAST使用方法一样，Diamond比对的第一步就是建库。Diamond的建库只支持蛋白质序列，需要你提供一个数据库的蛋白质fasta文件。为了方便大家的使用，小编给大家整理好了各种常用数据库的下载地址：

```
####NCBI-nr数据库下载
wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz

####env_nr(来自环境的nr）数据库下载
wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/env_nr.gz

####swissprot,高质量的蛋白数据库下载，蛋白序列得到实验的验证
wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/swissprot.gz

###通用蛋白质资源（UniProt90）
wget ftp://ftp.expasy.org/databases/uniprot/current_release/uniref/uniref90/uniref90.fasta.gz

```
下载好对应的数据库后，将其解压，就可以开始建库了。建库的方法也很简单需要使用makedb这个功能，这里我们以nr数据库为例：

```
diamond makedb --in nr.fa -d nr
```

建库结束后就可以开始diamond的运行了，最基本的使用方法也是相当简单:

```
diamond blastx -d nr -q reads.fna -o matches.m8
```

下面给大家讲讲，其它比较重要的参数，方便大家使用:

```
-threads/-p ###线程的使用数目，默认下diamond会自动探测当前环境下线程的数目，并全部使用
--sensitive ###精准的比对模式，默认情况下使用的是快速比对模式，通常建议用于比对更长的序列。默认模式主要用于短序列的比对，例如查找30-40aa片段上> 50位的重要匹配。
--outfmt/-f ###输出格式，这里常用的有两种，格式6的tabular分隔形式和格式5的XML格式。
--max-target-seqs/-k ####报告的每个比对查询的最大目标序列数
--evalue/-e ###输出报告中的e-value 值
```


###运行diamond时候，遇到的常见问题：

 **1. Diamond适合并行运行多个蛋白质fasta的比对吗？**


建议不要同时运行多个DIAMOND的任务在同一台机器上，因为如果将更多的资源分配给单个任务，效率其实会更高。


**2. Diamond比对好像不够准确，它没有找到我期待中的比对结果。**


有许多原因导致无法找到期待中的比对结果。原因如下:

 - 比对的hints可能没有满足最低的e-value的要求，及时他们是100%match的。
 - 低复杂度掩蔽和组成偏差校正(Low complexity masking and compositional bias correction)可能导致hints被过滤。
 - 由于算法的特点，diamond无法比对低于10AA蛋白质序列。另外，有一些高重复的input，会被过滤掉。

Diamond的基本介绍就到这里了，感兴趣的小伙伴赶紧拿起自己的数据去耍一耍，感受一下diamond快读比对的快感。到目前为止，根据我个人的使用体验，diamond的表现还是棒棒棒的。如果你也有相关的使用体验或者其它更快的工具推荐，欢迎在评论区一起讨论！！
 
参考资料：
1. Diamond github: https://github.com/bbuchfink/diamond
2. Diamond工具文章：http://www.nature.com/nmeth/journal/vaop/ncurrent/full/nmeth.3176.html 
 
  [1]: https://www.carats.io/wp-content/themes/crypterio/assets/images/DPA.png
  [2]: https://media.nature.com/lw926/nature-assets/nmeth/journal/v12/n1/images/nmeth.3176-F1.jpg
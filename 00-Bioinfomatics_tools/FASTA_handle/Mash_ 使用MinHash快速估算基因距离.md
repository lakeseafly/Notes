# Mash: 使用MinHash快速估算基因距离

标签（空格分隔）： 生信小工具

---

### 工具介绍

Mash扩展了MinHash降维技术，使其成对的突变距离和P值显着性检验，从而可以有效地聚类和搜索大量序列集合。混搭将大序列和序列集还原为较小的代表性草图，进而可以快速估计基因组之前全局突变距离。MASH能用于聚类NCBI RefSeq中所有的基因组，基于组装或者它的reads的实时的基因组聚类，聚类大量的宏基因组数据集。 该工具最终发表在Genome Biology上。

### 工具下载与基本使用

Github上很贴心，可以直接下载已经编译好的版本：

```
wget https://github.com/marbl/Mash/releases/download/v2.2/mash-Linux64-v2.2.tar
```
解压之后就能直接使用了
```
tar -zxvf mash-Linux64-v2.2.tar
cd mash-Linux64-v2.2/
./mash
```

我们简单看看其帮助文档：

```
Mash version 2.2

Type 'mash --license' for license and copyright information.

Usage:

  mash <command> [options] [arguments ...]

Commands:

  bounds    Print a table of Mash error bounds.

  dist      Estimate the distance of query sequences to references.

  info      Display information about sketch files.

  paste     Create a single sketch file from multiple sketch files.

  screen    Determine whether query sequences are within a larger mixture of
            sequences.

  sketch    Create sketches (reduced representations for fast operations).

  triangle  Estimate a lower-triangular distance matrix.
```

Mash 提供两个基本功能用于序列之间的比对 `sketch` 和 `dist`。 `sketch` 功能将序列转化为哈希结构图。  `dist` 功能比对 序列之前的sketches结果并且返回 Jaccard index的估计值,P值, 还有Mash距离, 这些值能用于估计一个简单进化模型的序列突变率。

## 具体使用例子

理论说了一大通，现在到大家最关键的环节，这个工具究竟可以怎么用？

### 例子一：基因组之间的遗传距离对比

下载两个Ecoil的测试数据：
```
wget https://gembox.cbcb.umd.edu/mash/genome1.fna
wget https://gembox.cbcb.umd.edu/mash/genome2.fna
```
由于基因组比较小，这里直接调用mash dist的功能：

```
mash dist genome1.fna genome2.fna
```

查看结果,不出意料两个基因组遗传距离非常接近：

```
###Reference-ID, Query-ID, Mash-distance, P-value, and Matching-hashes
genome1.fna   genome2.fna     0.0222766       0       456/1000
```


当然如果我们是要比对差别很大的，两个或者多个很大的基因组，直接使用dist功能就会很慢。这里mash的手册上推荐先使用sketch功能转化基因序列：

```
mash sketch genome1.fna
mash sketch genome2.fna
mash dist genome1.fna.msh genome2.fna.msh
```

除了基因组序列外，该功能还能直接应用到序列的reads中（fastq文件）。通过计算不同fastq文件之间的遗传距离，并将遗传距离结果整合可视化，你可以快速的得到你的不同个体测序序列之间的遗传关系：

![][1]

关于如何具体建树我这里就不细说，但是给大家推荐一个非常实用的Github网站，这个网站已经将Mash构树的过程封装好为mashtree，使用起来非常方便。Github链接：https://github.com/lskatz/mashtree


### 例子二：构建本地序列的mash数据库进行遗传距离分析

除了上面介绍的例子之外，mash还支持使用已知的序列构建本地的数据库，然后利用该构建的数据库，来对其它序列进行遗传距离的分析。

使用刚刚下载好的genome1.fna和genome2.fna进行本地mash数据库的构建：

```
mash sketch -o reference genome1.fna genome2.fna
```
构建好后可以使用`info`功能对构建好的数据库进行检查：

```
mash info reference.msh
###输出数据库的信息，证明数据库构成功：
Header:
  Hash function (seed):          MurmurHash3_x64_128 (42)
  K-mer size:                    21 (64-bit hashes)
  Alphabet:                      ACGT (canonical)
  Target min-hashes per sketch:  1000
  Sketches:                      2

Sketches:
  [Hashes]  [Length]  [ID]         [Comment]

  1000      4639675   genome1.fna  gi|49175990|ref|NC_000913.2| Escherichia coli str. K-12 substr. MG1655, complete genome

  1000      5498450   genome2.fna  gi|47118301|dbj|BA000007.2| Escherichia coli O157:H7 str. Sakai DNA, complete genome

```

下载新的序列，与构建好的mash数据库进行遗传距离比对：

```
wget https://gembox.cbcb.umd.edu/mash/genome3.fna
mash dist reference.msh genome3.fna

###结果显示genome3.fna与数据库的比对结果，可以发现genome1和genome3遗传具体是完全一致的（同一个样本）：
genome1.fna     genome3.fna     0       0       1000/1000
genome2.fna     genome3.fna     0.0222766       0       456/1000
```

### 例子三：与已构建好的RefSeq sketch数据库进行遗传距离比对

除了自己构建mash sketch数据库外，我们当然可以下载一些已经构建好的数据库。最常用的要数RefSeq sketch数据库。

下载好RefSeq数据库：

```
wget https://gembox.cbcb.umd.edu/mash/refseq.genomes.k21s1000.msh
```

由于mash没有给出测试文件，这里从我自己服务器中大豆的数据中抽取一部分进行测试：


```
head -n 1600 SRR7817178_1.fq > test_1.fq
head -n 1600 SRR7817178_2.fq > test_2.fq
cat test_1.fq test_2.fq >reads.fastq
```

这里使用`-m 2`参数来进一步提高结果的质量，这个参数会忽略掉低质量的单拷贝的k-mer：

```
mash sketch -m 2 reads.fastq
```
使用已下载好的RefSeq数据库，对序列的reads进行遗传相关系查询：

```
mash dist refseq.genomes.k21s1000.msh reads.fastq.msh > distances.tab
```
查看结果：

```
GCF_000004515.4_Glycine_max_v2.0_genomic.fna.gz reads.fastq     0.295981        2.36893e-05     1/1000
GCF_000297375.1_ASM29737v1_genomic.fna.gz       reads.fastq     0.295981        2.25234e-05     1/1000
GCF_000400935.1_ASM40093v1_genomic.fna.gz       reads.fastq     0.295981        2.16808e-05     1/1000
GCF_000400955.1_ASM40095v1_genomic.fna.gz       reads.fastq     0.295981        2.16542e-05     1/1000
GCF_000837185.1_ViralProj14097_genomic.fna.gz   reads.fastq     0.295981        1.63576e-05     1/1000
GCF_001411555.1_wgs.5d_genomic.fna.gz   reads.fastq     0.295981        2.36883e-05     1/1000
GCF_001578535.1_ASM157853v1_genomic.fna.gz      reads.fastq     0.295981        1.98188e-05     1/1000
GCF_001662445.1_ASM166244v1_genomic.fna.gz      reads.fastq     0.295981        2.31897e-05     1/1000
```
首先可以看到这些reads是和大豆(G.max)有较近的遗传距离，另外reads里面还含有一些污染，与其它细菌病毒有关系。

这里我们可以发现，mash是可以将reads的污染检测出来。事实上，mash提供了screen功能专门用于污染检测：

```
mash screen -w -p 4 refseq.genomes.k21s1000.msh|sort -gr |head
```

检测结果如下：

```
0.799067        9/1000  1       6.84538e-42     GCF_001343725.1_ViralMultiSegProj188731_genomic.fna.gz  [2 seqs] NC_020234.1 Rosellinia necatrix partitivirus 2 RdRp gene for RNA-dependent RNA polymerase, co                                                             mplete cds [...]
0.798763        6/672   1       2.34994e-28     GCF_000899295.1_ViralProj176433_genomic.fna.gz  NC_018671.1 Sauropus leaf curl disease associated DNA beta, complete genome
0.783786        6/1000  3       2.57053e-27     GCF_001401365.1_ViralProj299179_genomic.fna.gz  NC_028095.1 Torulaspora delbrueckii dsRNA Mbarr-1 killer virus strain EX1180 Kbarr-1 killer toxin gene, comple                                                             te cds
0.758374        2/666   1       2.73245e-09     GCF_000922435.1_ViralProj259986_genomic.fna.gz  NC_024777.1 Small begomovirus-associated satellite isolate Sa19-S1, complete sequence
0.758338        3/1000  1       2.27756e-13     GCF_000911175.1_ViralMultiSegProj225924_genomic.fna.gz  [2 seqs] NC_022616.1 Red clover cryptic virus 1 isolate IPP_Nemaro segment 1 RNA-dependent RNA polymer                                                             ase gene, complete cds [...]
0.743837        2/1000  27      6.16326e-09     GCF_001590135.1_ViralProj314638_genomic.fna.gz  NC_029628.1 Lake Sarah-associated circular virus-21 isolate LSaCV-21-LSMU-2013, complete sequence
0.743837        2/1000  2       6.16326e-09     GCF_000928835.1_ViralProj268860_genomic.fna.gz  NC_025790.1 Black sea bass polyomavirus 1 isolate 2835, complete genome
0.743837        2/1000  2       6.16326e-09     GCF_000888115.1_ViralProj45931_genomic.fna.gz   NC_013801.1 Croton yellow vein mosaic alphasatellite, complete genome
0.743837        2/1000  2       6.16326e-09     GCF_000848385.1_ViralProj14678_genomic.fna.gz   NC_001782.1 Saccharomyces cerevisiae killer virus M1, complete genome
0.743837        2/1000  2       6.16326e-09     GCF_000004515.4_Glycine_max_v2.0_genomic.fna.gz [1191 seqs] NC_016088.2 Glycine max cultivar Williams 82 chromosome 1, Glycine_max_v2.0, whole genome shotgun   
```

可以看到相对上面的结果，screen功能能够输出更加清晰的污染检测结果，包括了identity, shared-hashes, median-multiplicity, p-value, query-ID, query-comment等一系列的信息，更便于污染检测。

### 小结

mash的使用到这里就结束了，我尽量介绍了mash最常用的功能，更多的具体用法还需要大家回到其工作手册进行查询（原文链接）。总的来说，mash最大的优点就是速度快，使用方便，准确率高，是一款非常实用的小软件。


  [1]: http://static.zybuluo.com/lakesea/ai6tjvbfmkmvp0rukhn5kb73/%E6%8D%95%E8%8E%B711.PNG
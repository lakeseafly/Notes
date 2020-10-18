# 生信小工具：Plink之实战演练（2）

标签（空格分隔）： 生信工具

---

继续上次的内容，介绍完Plink与其基本格式后，当然就是到动真格的时候，实战演练，下面会通过例子和小伙伴们一起熟悉Plink的使用。

###测试数据的下载

下载官方教程中使用的测试软件：

```
wget http://zzz.bwh.harvard.edu/plink/hapmap1.zip
```

解压测试文件，看看里面有什么文件：

```
unzip hapmap1.zip
rm hapmap1.zip
ls -thrl
```
该测试集含了一下的四个文件：
```
-rw-rw-r-- 1 hhu hhu  1693668 Jun  6  2006 hapmap1.map
-rw-rw-r-- 1 hhu hhu 29739617 Jun  8  2006 hapmap1.ped
-rw-rw-r-- 1 hhu hhu      979 Jun  8  2006 pop.phe
-rw-rw-r-- 1 hhu hhu     1869 Jun  8  2006 qt.phe
```

简单介绍一下这些测试文件：这个hapmap**粗体文本**文件中包含了来自89个个体的随机选择的基因型（大约80,000个常染色体SNP）。**qt.phe:**数量性状在一个单独的替代表型文件中。 **pop.phe**文件包含一个虚拟表型，中国人个体编码为1，日本人编码为2。我们将用它来调查群体之间的差异。

###将map和ped格式的文件进行格式转换

在`--file`选项中map和ped格式文件的的前缀，这步会生成对应的bed,fam,bin格式的文件。
```
plink --file hapmap1
```
当该命令完成时，下面的信息也会出现，我们可以简单地了解到该变异文件含有83534和89个个体。
```
PLINK v1.90b6.3 64-bit (17 Jul 2018)
Options in effect:
  --file hapmap1

Hostname: zeus-1
Working directory: /scratch/pawsey0149/hhu/test/plink/new
Start time: Sun Jul 28 15:26:17 2019

Random number seed: 1564298777
257868 MB RAM detected; reserving 128934 MB for main workspace.
Scanning .ped file... done.
Performing single-pass .bed write (83534 variants, 89 people).
--file: plink.bed + plink.bim + plink.fam written.

End time: Sun Jul 28 15:26:18 2019
```
我们可以进一步压缩生成的文件为二进制的PED文件：

```
plink --file hapmap1 --make-bed --out hapmap1
```


例如，如果您想创建一个仅包含具有高频率基因分型（至少完整性为95％以上）的个体的新文件，我们可以运行：

```
plink --file hapmap1 --make-bed --mind 0.05 --out highgeno
```

###总结SNP变异的数据

####检查丢失率


准备工作做完后，我们开始统计一些SNP的数据。首先让我们先看看丢失率`missing rate`。

```
plink --bfile hapmap1 --missing --out miss_stat
```
这里要注意，因为上面我们已经将文件转为了二进制的bed格式，所以这里要用`--bfile`选项。

这里会生成两个文件：
```
-rw-r----- 1 hhu hhu 3.6M Jul 28 15:41 miss_stat.lmiss
-rw-r----- 1 hhu hhu 4.6K Jul 28 15:41 miss_stat.imiss
```

一个文件是`miss_stat.lmiss`这里包含着变异的missing rate信息，另一个文件是`miss_stat.imiss`包含着个提的missing rate信息。简单分布看看这两个文件，让大家更好的理解：

```
more miss_stat.lmiss

CHR          SNP   N_MISS     F_MISS
1    rs6681049        0          0
1    rs4074137        0          0
1    rs7540009        0          0
1    rs1891905        0          0
1    rs9729550        0          0
1    rs3813196        0          0
1    rs6704013        2  0.0224719
1    rs9439440        2  0.0224719
```

在这里展示了对于每个SNP，我们看到丢失个体的数量（N_MISS）和丢失个体的比例（F_MISS）。在让我们看看另外一个文件：

```
more miss_stat.imiss

    FID          IID MISS_PHENO     N_MISS     F_MISS
    HCB181            1          N        671 0.00803266
    HCB182            1          N       1156  0.0138387
    HCB183            1          N        498 0.00596164
    HCB184            1          N        412 0.00493212
    HCB185            1          N        329 0.00393852
    HCB186            1          N       1233  0.0147605
    HCB187            1          N        258 0.00308856

```

在这里展示了对于每个个体而已，我们看到丢失的SNP的总数量（N_MISS）和每个个体中丢失SNP占总SNP数目的比例（F_MISS)，一般来说如果该个体中丢失SNP的比例很高我们会在后续的分析之前选择移除该个体。在这里看来，这个测试数据质量还不错，个体的丢失SNP比例还是非常低的。

另外如果我们只对某一个染色体的丢失率有兴趣，我们也可以这样操作，指定输出对应染色体的丢失率信息：

```
plink --bfile hapmap1 --chr 1 --out res1 --missing
```


####检查等位基因频率

这里会生成一个freq_stat.frq文件，其中包含每个SNP的次要等位基因频率(MAF)。
```
plink --bfile hapmap1 --freq --out freq_stat
more freq_stat.frq

CHR         SNP   A1   A2          MAF  NCHROBS
   1   rs6681049    1    2       0.2135      178
   1   rs4074137    1    2      0.07865      178
   1   rs7540009    0    2            0      178
   1   rs1891905    1    2       0.4045      178
   1   rs9729550    1    2       0.1292      178
   1   rs3813196    1    2      0.02809      178
```

这里也可以对指定的SNP，检查其等位基因的频率

```
plink --bfile hapmap1 --snp rs1891905 --freq  --out snp1_frq_stat
```
既然这里讲到MAF，就简单扩展一下什么是MAF，为什么通过MAF进行过滤？

MAF是次要等位基因频率，即频率较低的等位基因的频率。可以出于多种原因对MAF进行过滤。其中之一是，在具有低覆盖量或深度的序列中，具有非常低MAF的样本可能会产生很多噪音，导致假阳性的下游分析。举一个极端的例子，仅存在于一个个体中的等位基因(非常低的等位基因频率)不会在群体结构推中添加任何有用可靠信息。。如果数据将用于全基因组关联研究（GWAS），通常要用比较高的MAF阈值，因为它需要非常强的统计能力来对非常罕见的等位基因做出有意义的陈述。一般常用的阀值有0.05和0.01。但是最佳MAF过滤阈值取决于你的数据类型，还有所计划进行的分析，数据质量和样本大小，可以通过统计分析计算出来。

这里给出一个例子，使用阀值0.05进行过滤：
```
plink --bfile hapmap1 -maf 0.05 -out test --make-bed

83534 variants loaded from .bim file.
89 people (89 males, 0 females) loaded from .fam.
89 phenotype values loaded from .fam.
Using 1 thread (no multithreaded calculations invoked).
Before main variant filters, 89 founders and 0 nonfounders present.
Calculating allele frequencies... done.
Total genotyping rate is 0.99441.
24114 variants removed due to minor allele threshold(s)
(--maf/--max-maf/--mac/--max-mac).
59420 variants and 89 people pass filters and QC.
Among remaining phenotypes, 44 are cases and 45 are controls.
--make-bed to test.bed + test.bim + test.fam ... done.
```
可以看到过滤后，24114 变异位点被过滤掉，59420变异位点被保留。

除了MAF以为`--mind`和`--geno`也是常见的过滤参数：

去除基因型丢失率大于10%的个体样本：
```
plink --file hapmap1 --recode --mind 0.10 --out hapmap1_complete
```
去除没有大于95%的个体都具有的变异位点：

```
plink --file hapmap1 --make-bed --geno 0.05
```

当然所有过滤其实也可以同时进行一步到位：

```
plink --file hapmap1 --make-bed --mind 0.10 --maf 0.05 --geno 0.05 --out hapmap1_clean
```

最后简单讲讲Plink也能用于LD的计算,更深入关于LD的计算以后再详细讨论：

```
plink -bfile hapmap1  -r2 -ld-window-kb 1 -ld-window 1000 -ld-window-r2 0 -out  test
```
当然有时候一些R包例如`adegenet`或者画structure图的structure，都需要特殊的输入文件格式（可输出的格式很多，这里只列举了一种，详细可支持回到plink的manual寻找），这时候Plink同时也可以帮上忙：

```
plink -bfile hapmap1 -aec -out man -recode A
```

到这里我就把我日常经常使用的Plink功能介绍了一下,但是这些功能其实只是Plink的冰山一角，其强大之处要你仔细阅读其manual才能深刻体会。


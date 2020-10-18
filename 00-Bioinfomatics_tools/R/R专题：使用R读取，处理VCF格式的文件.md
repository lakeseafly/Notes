# R专题：使用R读取，处理VCF格式的文件 

---
###什么是VCF？
VCF格式是用于描述SNP，INDEL还有SV结果的格式文件。了解如何读取，处理该格式文件，对处理变异的下游分析具有重要的意义。首先，先让大家通过一篇我们之前写的文章，回顾一下VCF格式文件的一些基本信息内容[VCF格式回顾][1]。


  [1]: https://mp.weixin.qq.com/s?__biz=MzUzMTEwODk0Ng==&mid=2247484335&idx=1&sn=d50e4694344aa07be17627925ea1d7c0&scene=21#wechat_redirect
  
  
###使用R处理VCF file

回顾完之后，就正式进入今天的主题。如何使用R处理VCF file。工欲善其事，必先利其器。处理VCF file也一样，只要选取恰当的R包，这并不是什么难事。接下来主要讲解运用VariantAnnotation 这个包进行VCF格式处理。当然这只是其中一种方式，一些新的或者其他的R包，例如vcfR也同样可以做到相应的处理。

####下载并且load进相应的包

首先，通过bioconductor下载VariantAnnotation这个R包。
```
source("https://bioconductor.org/biocLite.R")
biocLite("VariantAnnotation")
library(VariantAnnotation)
```

###Read
利用R包中reavcf的function读取用于测试的vcf文件

```
vcf<-readVcf("/home/ricky/Downloads/CEU.exon.2010_03.genotypes.vcf")
vcf
```

结果：
```
> vcf
class: CollapsedVCF 
dim: 3489 90 
rowRanges(vcf): 
  GRanges with 5 metadata columns: paramRangeID, REF, ALT, QUAL, FILTER （VCF中包括的tag）
info(vcf):
  DataFrame with 6 columns: DP, HM2, HM3, AA, AC, AN （数据库中包括的六列）
info(header(vcf)): （VCF文件中包括的header及其相应的解析）
       Number Type    Description                                                                                                 
   DP  1      Integer Total Depth                                                                                                 
   HM2 0      Flag    HapMap2 membership                                                                                          
   HM3 0      Flag    HapMap3 membership                                                                                          
   AA  1      String  Ancestral Allele, ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/pilot_data/technical/reference/ancestral_align...
   AC  1      Integer total number of alternate alleles in called genotypes                                                       
   AN  1      Integer total number of alleles in called genotypes                                                                 
geno(vcf):
  SimpleList of length 2: GT, DP
geno(header(vcf)):
      Number Type    Description               
   GT 1      String  Genotype                  
   DP 1      Integer Read Depth from MOSAIK BAM
```

####Read ROWDATA in VCF
快速浏览VCF里面文件的内容，简单给你一个6行，5列的overview：

```
> head(rowData(vcf))
DataFrame with 6 rows and 5 columns
  paramRangeID            REF                ALT      QUAL      FILTER
      <factor> <DNAStringSet> <DNAStringSetList> <numeric> <character>
1           NA              T                  C        NA        PASS
2           NA              G                  A        NA        PASS
3           NA              C                  T        NA        PASS
4           NA              T                  A        NA        PASS
5           NA              G                  A        NA        PASS
6           NA              T                  C        NA        PASS
```

只查看有数据的前三行的数据

```
> head(fixed(vcf),3)
DataFrame with 3 rows and 4 columns
             REF                ALT      QUAL      FILTER
  <DNAStringSet> <DNAStringSetList> <numeric> <character>
1              T                  C        NA        PASS
2              G                  A        NA        PASS
3              C                  T        NA        PASS
```


####Read ALT in VCF

只读取有发生变异的base的那一列：
```
> alternate <- alt(vcf)
> alternate
DNAStringSetList of length 3489 （一共有3489个变异位点）
[[1]] C
[[2]] A
[[3]] T
[[4]] A
[[5]] A
[[6]] C
[[7]] C
[[8]] T
[[9]] A
[[10]] A
...
<3479 more elements>
```


####Read GENO in VCF

现在讲讲如何提取Genotype的信息。首先查看该VCF中含有geno的信息有什么:
```
> geno(vcf)
List of length 2
names(2): GT DP
```
这个文件只有两个信息 GT还有DP。通常其他的VCF file中还会含有GQ GL SP PL等相关的信息。

```
> geno(vcf)$GT[1:2,1:15]
              NA06984 NA06985 NA06986 NA06989 NA06994 NA07000 NA07037 NA07048 NA07051 NA07346 NA07347 NA07357 NA10847 NA10851
1:1105366_T/C "."     "."     "0/0"   "."     "."     "0/0"   "0/0"   "0/0"   "0/0"   "0/0"   "0/0"   "0/0"   "."     "0/0"  
1:1105411_G/A "."     "."     "0/0"   "."     "."     "0/0"   "0/0"   "0/0"   "1/0"   "0/0"   "0/0"   "0/0"   "."     "0/0"  
              NA11829
1:1105366_T/C "0/0"  
1:1105411_G/A "0/0"  

```

```
> geno(vcf)$DP[1:2,1:15]
              NA06984 NA06985 NA06986 NA06989 NA06994 NA07000 NA07037 NA07048 NA07051 NA07346 NA07347 NA07357 NA10847 NA10851
1:1105366_T/C       0       0     107       0       0      25      30      31      57      69      53     225       0       6
1:1105411_G/A       0       0      92       0       0      23      17      37      61      60      47     126       0       5
              NA11829
1:1105366_T/C      79
1:1105411_G/A      79
```



####Read Subset of VCF
如果你只想读取VCF中信息的一部分，你还可以用ScanVcfParam对你想选择的信息进行相应的提取。
```
> svp <- ScanVcfParam(fixed="ALT", info="DP", geno="GT")
> vcf_flds <- readVcf("/home/ricky/Downloads/CEU.exon.2010_03.genotypes.vcf", "test", svp)
> geno(vcf_flds)
List of length 1
names(1): GT
> vcf_flds
class: CollapsedVCF 
dim: 3489 90 
rowRanges(vcf):
  GRanges with 3 metadata columns: paramRangeID, REF, ALT
info(vcf):
  DataFrame with 1 column: DP
info(header(vcf)):
      Number Type    Description
   DP 1      Integer Total Depth
geno(vcf):
  SimpleList of length 1: GT
geno(header(vcf)):
      Number Type   Description
   GT 1      String Genotype   
```

用R对VCF格式的文件进行读取和基本的操作就介绍到这里，这只是比较基础简单的一部分，如果大家还想继续深入探究可以阅读相关R包的manual：vcfR和VariantAnnotation。
# 群体遗传专题：关于SNP的过滤（1）：如何使用vcftools进行SNP过滤 

标签（空格分隔）： 群体遗传

---


###前言
> 很多关注我这个专题的朋友都在后台留言，说想了解更多关于SNP过滤的东西。是的，确实这是一个比较重要的问题。第一，过滤到一些低质量的SNP可以防止calling一些假阳性的SNP，这些假阳性的SNP会很大程度影响到后续的一系列的分析，例如GWAS等的分析，最后影响相关生物学问题的解答。第二，如果你有很多的个体，往往你的call完SNP后，VCF文件的大小的会比较大，如果不经过过滤，对下游的群体结构分析，PCA等相关的计算，都会对计算机产生巨大的负担（运行速度还有需要的内存等）。讲了那么多，在接下来几周会从简单开始，讨论学习一下SNP的过滤的流程，常用工具，还有通常推荐使用的一些参数。



文章主要是根据网上SNP Filtering Tutorial的课程总结翻译而成，推荐新手跟着一步一步来操作，进而理解掌握。老司机就自行忽略看看就好了。

###预备工作
创建文件夹，下载测试数据包，解压文件

```
mkdir filtering
cd filtering
curl -L -o data.zip https://www.dropbox.com/sh/bf9jxviaoq57s5v/AAD2Kv5SPpHlZ7LC7sBz4va8a?dl=1
unzip data.zip 
ls -thrl 
```

看看数据包里包含了什么文件

```
total 73742
-rwxr--r-- 1 hhu pawsey0149   109633 Mar  6  2015 BR_004-RG.bam
-rwxr--r-- 1 hhu pawsey0149   247496 Mar  6  2015 BR_004-RG.bam.bai
-rwxr--r-- 1 hhu pawsey0149   120045 Mar  6  2015 BR_006-RG.bam
-rwxr--r-- 1 hhu pawsey0149   247712 Mar  6  2015 BR_006-RG.bam.bai
drwxr-s--- 2 hhu pawsey0149     4096 Jul  1 14:54 __MACOSX
-rwxr--r-- 1 hhu pawsey0149      399 Mar  6  2015 popmap
-rwxr--r-- 1 hhu pawsey0149 68264393 Mar  6  2015 raw.vcf.gz
-rwxr--r-- 1 hhu pawsey0149  6804314 Mar  6  2015 reference.fasta
-rwxr--r-- 1 hhu pawsey0149  1085387 Mar  6  2015 reference.fasta.amb
-rwxr--r-- 1 hhu pawsey0149  1068884 Mar  6  2015 reference.fasta.ann
-rwxr--r-- 1 hhu pawsey0149  6379720 Mar  6  2015 reference.fasta.bwt
-rwxr--r-- 1 hhu pawsey0149   353544 Mar  6  2015 reference.fasta.clstr
-rwxr--r-- 1 hhu pawsey0149   976388 Mar  6  2015 reference.fasta.fai
-rwxr--r-- 1 hhu pawsey0149  1594913 Mar  6  2015 reference.fasta.pac
-rwxr--r-- 1 hhu pawsey0149  3189872 Mar  6  2015 reference.fasta.sa
-rwxr--r-- 1 hhu pawsey0149   137209 Mar  6  2015 stats.out
```

下载并且安装VCFTOOLS

```
git clone https://github.com/vcftools/vcftools.git
cd vcftools
./autogen.sh
./configure
make
make install
```

该文章也会用到一个叫做mawk的软件，它其实就是更加简单版的awk，也需要下载，我这里偷懒用conda下载，其实上面的vcftools 也可以用conda下载
```
conda install mawk
```

###小试牛刀

首先做一个过滤，
我们将只保留已那些变异，至少50%的个体成功被snp calling的位点，最低质量分数为30的位点，次要等位基因计数为3的位点。

```
vcftools --gzvcf raw.vcf.gz --max-missing 0.5 --mac 3 --minQ 30 --recode --recode-INFO-all --out raw.g5mac3
```
解读一下这个代码做了什么：
在这段代码中，我们调用vcftools，在--vcf标志后给它一个vcf文件， -  max-missing 0.5告诉它过滤低于50％的基因型（跨所有个体）--mac 3标志告诉它过滤次要等位基因计数小于3的SNP。
这与基因型有关，因此必须在至少1个纯合子和1个杂合子或3个杂合子中的变异调用。 --recode标志告诉程序使用过滤器写入一个新的vcf文件，--recode-INFO-all保留旧vcf文件中的所有INFO标志。--out 输出的文件名称

运行完后，在屏幕或者生成的log file中会有这样的信息：

```
After filtering, kept 40 out of 40 Individuals
Outputting VCF file...
After filtering, kept 78434 out of a possible 147540 Sites
Run Time = 17.00 seconds
```

这个output给了你一些基本的信息，这里你所用到的40个个体，全部被保留下来，然后147540中的78434个变异位点经过过滤后被保留下来（几乎一半的变异位点被过滤掉）。运行时间：17秒。

在得到这个过滤完的文件后，继续进一步过滤：

我们接下来过滤的标准是基因型调用的最小深度和最小平均深度。

```
vcftools --vcf raw.g5mac3.recode.vcf --minDP 3 --recode --recode-INFO-all --out raw.g5mac3dp3 
```

运行完的log file 结果如下：

```
After filtering, kept 40 out of 40 Individuals
Outputting VCF file...
After filtering, kept 78434 out of a possible 78434 Sites
Run Time = 14.00 seconds
```
可以惊讶的发现，所有的位点都被保留了下来。这说明，上述第一步得到的过滤的文件中的所有位点，已经满足了最小深度至少大于3的过滤条件。

在这里你可能会提出疑问，最小深度大于3这个条件是否足够严谨filter出置信的SNP （理论上是越高越好，但是过高也不太好，这个内容会再以后的文章再具体讨论）。简短的回答是像FreeBayes和GATK这样复杂的多样本SNP calling的工具可以自信地调用几乎没有读数的基因型，因为所有样本的变异是同时评估预测的。因此，这里的过滤是基于来自所有个体的所有读数的三个reads和还有其他的先验信息。但是，我们还会做很多其他的过滤步骤！

如果你还是担心深度不够的影响，我们也可以测测低深度可能产生的SNP错误率。

下载好写好的脚本
```
curl -L -O https://github.com/jpuritz/dDocent/raw/master/scripts/ErrorCount.sh
chmod +x ErrorCount.sh 
./ErrorCount.sh raw.g5mac3dp3.recode.vcf 
```


该脚本计算由于低读取深度而导致的潜在基因分型错误的数量，它会报告基于观察杂合子中第二等位基因的50％二项概率和基于25％概率的高范围。

运行结果
```
Potential genotyping errors from genotypes from only 1 read range from 0 to 0.0
Potential genotyping errors from genotypes from only 2 reads range from 0 to 0.0
Potential genotyping errors from genotypes from only 3 reads range from 15986 to 53714.22
Potential genotyping errors from genotypes from only 4 reads range from 6230 to 31502.04
Potential genotyping errors from genotypes from only 5 reads range from 2493 to 18914
40 number of individuals and 78434 equals 3137360 total genotypes
Total genotypes not counting missing data 2380094
Total potential error rate is between 0.0103815227466 and 0.0437504821238
SCORCHED EARTH SCENARIO
WHAT IF ALL LOW DEPTH HOMOZYGOTE GENOTYPES ARE ERRORS?????
The total SCORCHED EARTH error rate is 0.129149100834.
```

现在，由于基因型少于5的深度，VCF文件的最大错误率小于5％。看，没什么好担心的。

下一步，我们可以去除那些没有被测序的很好的个体，我们可以通过估计个体间数据丢失的水平来过滤。

```
vcftools --vcf raw.g5mac3dp3.recode.vcf --missing-indv
```

这会生成一个out.imiss的文件，看看他里面是什么：
```
cat out.imiss

INDV	N_DATA	N_GENOTYPES_FILTERED	N_MISS	F_MISS
BR_002	78434	0	13063	0.166548
BR_004	78434	0	16084	0.205064
BR_006	78434	0	25029	0.319109
BR_009	78434	0	30481	0.38862
BR_013	78434	0	69317	0.883762
BR_015	78434	0	8861	0.112974
BR_016	78434	0	29789	0.379797
BR_021	78434	0	17422	0.222123
BR_023	78434	0	43913	0.559872
BR_024	78434	0	24220	0.308795
BR_025	78434	0	21998	0.280465
BR_028	78434	0	26786	0.34151
BR_030	78434	0	74724	0.952699
BR_031	78434	0	26488	0.337711
BR_040	78434	0	19492	0.248515
BR_041	78434	0	17107	0.218107
BR_043	78434	0	16384	0.208889
BR_046	78434	0	28770	0.366805
BR_047	78434	0	13258	0.169034
BR_048	78434	0	24505	0.312428
WL_031	78434	0	22566	0.287707
WL_032	78434	0	22604	0.288191
WL_054	78434	0	32902	0.419486
WL_056	78434	0	34106	0.434837
WL_057	78434	0	37556	0.478823
WL_058	78434	0	31448	0.400949
WL_061	78434	0	35671	0.45479
WL_064	78434	0	47816	0.609634
WL_066	78434	0	10062	0.128286
WL_067	78434	0	47940	0.611215
WL_069	78434	0	38260	0.487799
WL_070	78434	0	21188	0.270138
WL_071	78434	0	16692	0.212816
WL_072	78434	0	46347	0.590904
WL_076	78434	0	78178	0.996736
WL_077	78434	0	55193	0.703687
WL_078	78434	0	54400	0.693577
WL_079	78434	0	19457	0.248068
WL_080	78434	0	30076	0.383456
WL_081	78434	0	30334	0.386746

```


您可以看到有些人的数据丢失率高达99.6％。我们绝对想要过滤掉这些。现在我们需要创建一个缺失数据超过50％的个体的列表。

```
mawk '$5 > 0.5' out.imiss | cut -f1 > lowDP.indv
```

现在我们有一个要删除的个体列表，我们可以直接将其提供给VCFtools进行过滤。

```
vcftools --vcf raw.g5mac3dp3.recode.vcf --remove lowDP.indv --recode --recode-INFO-all --out raw.g5mac3dplm
```

然后该文章的原作者非常贴心的，


我们可以将变异限制为在高百分比的个体中，并通过基因型的平均深度进行过滤。
```
vcftools --vcf raw.g5mac3dplm.recode.vcf --max-missing 0.95 --maf 0.05 --recode --recode-INFO-all --out DP3g95maf05 --min-meanDP 20
```


这对所有个体应用的calling rate 为（95％）。这对群体是采集自两个地方的变异来说是已足够。但是当你有多个采样地点时，你还希望按个体特定的calling rate进行过滤。 VCFtools不会直接计算，但这是一个简单的解决方法。首先，我们需要一个文件来定义地点（个体）。大多数程序希望文件具有两个制表符分隔列。首先是样本名称，第二个是个体的群体类型分配。

```
cat popmap

BR_002	BR
BR_004	BR
BR_006	BR
BR_009	BR
BR_013	BR
BR_015	BR
BR_016	BR
BR_021	BR
BR_023	BR
BR_024	BR
BR_025	BR
BR_028	BR
BR_030	BR
BR_031	BR
BR_040	BR
BR_041	BR
BR_043	BR
BR_046	BR
BR_047	BR
BR_048	BR
WL_031	WL
WL_032	WL
WL_054	WL
WL_056	WL
WL_057	WL
WL_058	WL
WL_061	WL
WL_064	WL
WL_066	WL
WL_067	WL
WL_069	WL
WL_070	WL
WL_071	WL
WL_072	WL
WL_076	WL
WL_077	WL
WL_078	WL
WL_079	WL
WL_080	WL
WL_081	WL
```

现在我们需要创建两个列表，这些列表只包含每个个体的个别名称。

```
mawk '$2 == "BR"' popmap > 1.keep && mawk '$2 == "WL"' popmap > 2.keep
```


接下来，我们使用VCFtools来估计每个群体中基因座的缺失数据。

```
vcftools --vcf DP3g95maf05.recode.vcf --keep 1.keep --missing-site --out 1
vcftools --vcf DP3g95maf05.recode.vcf --keep 2.keep --missing-site --out 2 
```
这会生成两个文件分别是 1.lmiss 和2.lmiss

快速看看文件里面的内容：
```
head -3 1.lmiss

CHR     POS     N_DATA  N_GENOTYPE_FILTERED     N_MISS  F_MISS
E1_L101 9       34      0                       0       0
E1_L101 15      34      0                       0       0
```
这里文章的原作者在第一行加了header 让你更容易去理解每一列。但我们感兴趣的是最后一列是该基因座缺失数据的百分比。我们可以合并这两个文件，并制作一个关于要删除的10％缺失数据阈值的基因座列表。请注意，这是丢失数据总体速率的两倍。

```
cat 1.lmiss 2.lmiss | mawk '!/CHR/' | mawk '$6 > 0.1' | cut -f1,2 >> badloci
```


然后，我们将此文件反馈到VCFtools以删除任何基因座。

```
vcftools --vcf DP3g95maf05.recode.vcf --exclude-positions badloci --recode --recode-INFO-all --out DP3g95p5maf05
```


同样，我们只有两个个体，所以我们的过滤可以过滤这些。然而，在多地点采样的研究就不能像这样处理了。 
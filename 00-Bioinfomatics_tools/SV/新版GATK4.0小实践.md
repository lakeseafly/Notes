# 新版GATK4.0小实践

标签（空格分隔）： 学习小组

GATK升级4.0版了，作为一个call variant的非常常用的软件，之前我一直是在使用3.8的版本，这次也正好整理下，将GATK基本分析流程过渡到4.0版本，看看具体有什么异同。

GATK4.0最明显的变化是其命令调用发生了改变，可以看看这个就明白了https://software.broadinstitute.org/gatk/documentation/quickstart

GATK4.0还集成了picard工具以及增加了SortSam功能，这样Germline short variant discovery流程只要GATK一个软件即可完成sam/bam文件到vcf文件全流程。

整体上Germline call variant分析思路没有变，我按照Germline short variant discovery (SNPs + Indels)教程上所示流程图，将GATK4.0基本用法过一遍，并且同时使用samtools/bcftools call variant，最后整理出两个工具共有的SNP。由于我使用的数据是真菌，没有黄金的变异参考位点，所以这里就跳过了BQSR这一步。整体的流程做植物做真菌的同学都可以参考，做人的话就要更加细致一点。多加BQSR等步骤。


###质控
第一次使用fastp对测序数据质控，非常好用的一个质控软件，海普洛斯开发的，速度和效果都蛮好，在检测完质量后，马上对数据进行了trimming的操作，很方便。
https://github.com/OpenGene/fastp
```
fastp -i V1.1.fastq -I V1.2.fastq -o V1.1.clean.fastq -O V1.2.clean.fastq -w 4
fastp -i V2.1.fastq -I V2.2.fastq -o V2.1.clean.fastq  -O V2.2.clean.fastq -w 4
```
屏幕输出结果如下
```
Read1 before filtering:
total reads: 500000
total bases: 61376701
Q20 bases: 58780405(95.7699%)
Q30 bases: 55609802(90.6041%)

Read1 after filtering:
total reads: 499134
total bases: 61282161
Q20 bases: 58701825(95.7894%)
Q30 bases: 55546088(90.6399%)

Read2 before filtering:
total reads: 500000
total bases: 59514718
Q20 bases: 55985085(94.0693%)
Q30 bases: 52065924(87.4841%)

Read2 aftering filtering:
total reads: 499134
total bases: 59455581
Q20 bases: 55949906(94.1037%)
Q30 bases: 52045738(87.5372%)

Filtering result:
reads passed filter: 998268
reads failed due to low quality: 1732
reads failed due to too many N: 0
reads failed due to too short: 0
reads with adapter trimmed: 170
bases trimmed due to adapters: 7206

Duplication rate: 17.7822%

Insert size peak (evaluated by paired-end reads): 165
```

###比对(Mapping)

这里使用官网推荐的bwa进行比对
**Index** 
```
bwa index genome.fasta
```
**Alignment**
这里一定要把生成的BAMfile文件进行ID，LIB等信息的记录，最后将样本名字用上，使用-R option，这个信息在后面call variant中用以区分这些bam file是来自哪一个个体。
```
bwa mem -M -t 10 -R @RG\tID:V1\tLB:V1\tPL:ILLUMINA\tPM:HISEQ\tSM:V1 genome.fasta V1.1.clean.fastq V1.2.clean.fastq
bwa mem -M -t 10 -R @RG\tID:V2\tLB:V2\tPL:ILLUMINA\tPM:HISEQ\tSM:V2 genome.fasta V2.1.clean.fastq V2.2.clean.fastq
```
**Sam2sort_bam**
将sam格式转化为bam格式，并且排序，这一步现在能用gatk这届做了
```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar SortSam -I V1.sam -O V1.sort.bam -SO coordinate --CREATE_INDEX true
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar SortSam -I V2.sam -O V2.sort.bam -SO coordinate --CREATE_INDEX true
```

**Marked PCR duplicated regions**
标记PCR重复序列，现在也能用gatk直接做了

```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar MarkDuplicates -I V2.sort.bam -O V2.sort.marked.bam -M V2.metrics
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar MarkDuplicates -I V1.sort.bam -O V1.sort.marked.bam -M V1.metrics
```
**Fixmate**
mate-pairde rads 一般有比较长的insert size，如果没有修正或者去除会影响到vairant calling，验证mate-pair信息并在需要时修复,这一步是可选的一步，GATK中并没说一定需要做。
```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar FixMateInformation -I V1.sort.marked.bam -O V1.sort.marked.fix.bam -SO coordinate --CREATE_INDEX true
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar FixMateInformation -I V2.sort.marked.bam -O V2.sort.marked.fix.bam -SO coordinate --CREATE_INDEX true
```


**Index** 
在运行variant calling之前，GATK需要对ref进行index
```
samtools faidx genome.fasta
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar CreateSequenceDictionary -R genome.fasta -O genome.dict
```
**Variant calling**

使用Hapyotercaller进行variant calling，使用HaplotypeCaller对上述的bam文件进行call variant，就是寻找变异的过程。对群体来call，一般使用 -ERC GVCF,先生成初始的gvcf。

```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar HaplotypeCaller -R genome.fasta -I V1.sort.marked.fix.bam -ERC GVCF -O V1.g.vcf 
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar HaplotypeCaller -R genome.fasta -I V1.sort.marked.fix.bam -ERC GVCF -O V1.g.vcf 
```

升级到4.0后，GenotypeGVCFs既不在支持多个VCF的输入了（只支持一个），所以这里先使用combinegvcfs，然后在做GenotypeGVCFs。


```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar CombineGVCFs -R genome.fasta -V V1.g.vcf -V V2.g.vcf -O V.vcf 

java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar GenotypeGVCFs -R genome.fasta -V V.vcf -O variants.gatk.raw1.vcf
```

与此同时也使用samtools进行variant calling，并index其生成的文件:

```
samtools mpileup -t AD,ADF,ADR,DP,SP -ugf genome.fasta V1.sort.marked.fix.bam V2.sort.marked.fix.bam | bcftools call -vm > variants.samtools.raw1.vcf
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar IndexFeatureFile --feature-file variants.samtools.raw1.vcf
```

寻找出两个不同工具call 出来variant相同部分:

```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar SelectVariants -R genome.fasta --variant variants.gatk.raw1.vcf --concordance variants.samtools.raw1.vcf -O variants.concordance.raw1.vcf 
```

并对去进行hard filtering，使用参数进行过滤:

```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar VariantFiltration -R genome.fasta -V variants.concordance.raw1.vcf --filter-name FilterQual --filter-expression "QUAL < 60.0" --filter-name FilterQD --filter-expression "QD < 20.0" --filter-name FilterFS --filter-expression "FS > 13.0" --filter-name FilterMQ --filter-expression "MQ < 30.0" --filter-name FilterMQRankSum --filter-expression "MQRankSum < -1.65" --filter-name FilterReadPosRankSum --filter-expression "ReadPosRankSum < -1.65" -O variants.concordance.flt1.vcf
grep -vP "\tFilter" variants.concordance.flt1.vcf > variants.concordance.filter1.vcf

```


最后将SNPs和INDEL摘出来:
```
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar SelectVariants -R genome.fasta -V variants.concordance.filter1.vcf --select-type-to-include INDEL -O INDEL.vcf
java -jar gatk-4.0.8.1/gatk-package-4.0.8.1-local.jar SelectVariants -R genome.fasta -V variants.concordance.filter1.vcf --select-type-to-include SNP -O SNP.vcf
```

整个流程就差不多到这，初次体验感觉GATK4.0大部分基本都和3.8的操作类似，只是有个别的option换了不同的代表符。在4.0中取消了很多可以使用多个线程的选项，只保留下Hapyotercaller可以选择多个线程`(--native-pair-hmm-threads)`。4.0应该是采用了spark进行多个线程的管理，但由于测试数据比较小，就未采用，具有应用方面还需要以后继续探究。


###变异检测那么简单吗？
经过简单的实战之后，似乎变异检测是一件非常容易的事情，只要敲几行代码就行了。当然这只是最表面的一些东西，当你深入了解这些工具的背后，还有相关的原理算法之后，你就会发现事情并没有那么简单。

由于下面的种种因素：

 - DNA序列可以非常的长
 - A/T/G/C能够构建任意组合的DNA序列，因此在完全随机情况下，即使随机分配也能产生各种各样的模式。
 - DNA序列只有部分会受到随机影响，基本上这部分序列都是有功能的。
 - 不同物种的不同的DNA序列受到不同的随机性影响
 - 按照实验流程将大片段DNA破碎成小的部分，并尝试通过和参考基因组比对找到原来的位置。只有它依旧和原来的位置非常靠近，才能进一步寻找变异。

因此即便序列和基因组某个序列非常接近，从算法的角度是正确比对，但其实偏离了原来正确的位置，那么从这部分找到的变异也是错误的。那我们有办法解决这个问题嘛？基本上不可能，除非技术进步后，我们可以一次性通读所有序列。当然目前比较常用的方法是找到最优变异，且那个变异能更好的解释问题，使每条read中的变异都是一致的。GATK4.0 中Hapyotercaller的算法一般可以很好地解决该问题。相对旧点的软件，常用策略：**realignment**或**probabilistic alignment**。
 
参考文章：
https://www.plob.org/article/11388.html

----------
**轻松一刻**
给大家分享一副很可爱的系统进化树图：
**嘿嘿嘿**，没想到恐龙竟然在鳄鱼还有其他鸟禽类之间。
![][1]
 
 
 



  [1]: http://static.zybuluo.com/lakesea/zekwetg1a4mbyz00vbfafr9g/DrLv1f7U0AAfCnl.jpg
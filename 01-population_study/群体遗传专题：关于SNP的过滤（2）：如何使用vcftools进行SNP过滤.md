# 群体遗传专题：关于SNP的过滤（2）：如何使用vcftools进行SNP过滤 

标签（空格分隔）： 群体遗传

---

> 继续上一节课教程的下半节，继续介绍一些其他过滤SNP的工具，欢迎大家跟着学习。


从现在开始，过滤步骤假定vcf文件是由FreeBayes生成的。（但是其他的vcf也是类似的，可能在header等不同地方有小许的差异）。

###查看VCF 文件的Header

FreeBayes在VCF文件中输出大量关于基因座的信息，使用这些信息和RADseq的属性，我们为数据添加了一些复杂的过滤器。让我们看看我们的VCF文件的header，并快速查看所有信息。

```
mawk '/#/' DP3g95maf05.recode.vcf
```

这将输出几行INFO标签，我在下面展示了抽取的一部分信息：
```
##fileformat=VCFv4.1
##fileDate=20150301
##source=freeBayes v0.9.20-16-g3e35e72
##reference=reference.fasta
##phasing=none
##INFO=<ID=NS,Number=1,Type=Integer,Description="Number of samples with data">
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total read depth at the locus">
##INFO=<ID=DPB,Number=1,Type=Float,Description="Total read depth per bp at the locus; bases in reads overlapping / bases in haplotype">
##INFO=<ID=AC,Number=A,Type=Integer,Description="Total number of alternate alleles in called genotypes">
##INFO=<ID=AN,Number=1,Type=Integer,Description="Total number of alleles in called genotypes">
```

###vcffilter的安装和使用

vcffilter是基于等位基因平衡的一个工具。那什么是等位基因平衡？0到1之间的数字，表示参考等位基因的reads与所有reads的比率。在仅考虑来自被称为杂合子的个体的reads情况下，我们期望我们的数据中的等位基因应该接近0.5。这样情况下，我们可以使用vcflib的vcffilter程序。 


可以快速用conda 安装

```
conda install vcflib
```
或者自行到GitHub下载安装 (https://github.com/ekg/vcflib)


快速了解一下vcffilter的使用说明

```
vcffilter

usage: vcffilter [options] <vcf file>

options:
    -f, --info-filter     specifies a filter to apply to the info fields of records,
                          removes alleles which do not pass the filter
    -g, --genotype-filter specifies a filter to apply to the genotype fields of records
    -k, --keep-info       used in conjunction with '-g', keeps variant info, but removes genotype
    -s, --filter-sites    filter entire records, not just alleles
    -t, --tag-pass        tag vcf records as positively filtered with this tag, print all records
    -F, --tag-fail        tag vcf records as negatively filtered with this tag, print all records
    -A, --append-filter   append the existing filter tag, don't just replace it
    -a, --allele-tag      apply -t on a per-allele basis.  adds or sets the corresponding INFO field tag
    -v, --invert          inverts the filter, e.g. grep -v
    -o, --or              use logical OR instead of AND to combine filters
    -r, --region          specify a region on which to target the filtering, requires a BGZF
                          compressed file which has been indexed with tabix.  any number of
                          regions may be specified.
```
###第一次使用vcffilter过滤尝试


```
vcffilter -s -f "AB > 0.25 & AB < 0.75 | AB < 0.01" DP3g95maf05.recode.vcf > DP3g95p5maf05.fil1.vcf
```


vcffilter使用简单的条件语句，因此过滤掉小于等于0.25且高于0.75的等位基因平衡的基因座。但是，它确还是包括了等位基因平衡接近于0的基因座。最后一个条件是捕获那些具有固定变异的基因座（所有个体对于两个变体之一是纯合的）。使用 ```-s```告诉过滤工具应用于sites每个位点，而不仅仅是等位基因使用。



要查看VCF文件中现在有多少个基因座，您只需使用简单的mawk语句。

```
mawk '!/#/' DP3g95p5maf05.recode.vcf | wc -l
12754 
#过滤前基因座的数量
```

```
mawk '!/#/' DP3g95p5maf05.fil1.vcf | wc -l
9678
#过滤后基因座的数量
```


您会注意到我们已经过滤了很多基因座。根据我的经验，我发现其中大多数往往是某种错误导致的。但是，这将取决于数据。我建议您探索自己的数据集。

###下一个过滤练习


下一个过滤练习，我们过滤掉两个链都有reads都有的位点。对于GWAS甚至RNAseq，这可能是一件好事。
除非您使用超小基因组片段或长reads（MiSeq）。 SNP应仅由正向reads或仅反向reads涵盖。

```
vcffilter -f "SAF / SAR > 100 & SRF / SRR > 100 | SAR / SAF > 100 & SRR / SRF > 100" -s DP3g95p5maf05.fil1.vcf > DP3g95p5maf05.fil2.vcf
```


这次的过滤是基于比例，因此一些无关的reads不会删除整个基因座。简单来说，它保留的位点比反向交替reads的前向交替reads多100倍，前向参考reads比反向参考reads多100倍。

查看一下剩下基因座的数量
···
mawk '!/#/' DP3g95p5maf05.fil2.vcf | wc -l
9491
···

这只能去除一小部分基因座，但这些基因座可能是旁系同源物，微生物污染物或奇怪的PCR嵌合体。（对比DP3g95p5maf05.fil1.vcf, 这次只过滤了不到200个位点 ）


samtools是直接从终端可视化比对的好方法

```
samtools tview BR_006-RG.bam reference.fasta -p E28188_L151 
#-p 具体展示那个位置的比对情况
```
这里由于微信公众号的显示，不能很好地展示，就不放出来了，建议大家自己亲自尝试，以获取更好的理解。


下一个过滤标准是根据参考和备用等位基因之间的比对质量比率来过滤

```
vcffilter -f "MQM / MQMR > 0.9 & MQM / MQMR < 1.05" DP3g95p5maf05.fil2.vcf > DP3g95p5maf05.fil3.vcf
```



这里过滤的基本原理是，因为RADseq基因座和等位基因都应该从相同的基因组位置开始，所以两个等位基因的作图质量之间不应该存在很大的差异。看看剩余的基因座数目：

```
mawk '!/#/' DP3g95p5maf05.fil3.vcf | wc -l
9229 
```


这过滤掉了不到3％的变体，但可能还需要进行过滤。
继续过滤，根据它们是否支持参考或备选等位基因的reads的正确配对状态的差异进行过滤。

```
vcffilter -f "PAIRED > 0.05 & PAIREDR > 0.05 & PAIREDR / PAIRED < 1.75 & PAIREDR / PAIRED > 0.25 | PAIRED < 0.05 & PAIREDR < 0.05" -s DP3g95p5maf05.fil3.vcf > DP3g95p5maf05.fil4.vcf
```


由于从头组装并不完美，因此某些基因座只会将unpaired的reads比对到它们。这不是问题。当支持参考等位基因的所有reads是paired但不支持备用等位基因时，会出现问题，所以要将其过滤。

```
mawk '!/#/' DP3g95p5maf05.fil4.vcf | wc -l
9166
```
可以看到，我们的位点数量逐渐减少，但我们的信噪比不断增加。

###额外的过滤步骤

因为reads不是随机分布在contigs上的，所以可以进一步过滤，首先：删除任何质量得分低于深度1/4的基因座。

```
vcffilter -f "QUAL / DP > 0.25" DP3g95p5maf05.fil4.vcf > DP3g95p5maf05.fil5.vcf
```


第二步更复杂。先创建每个基因座深度的列表：
```
cut -f8 DP3g95p5maf05.fil5.vcf | grep -oe "DP=[0-9]*" | sed -s 's/DP=//g' > DP3g95p5maf05.fil5.DEPTH
```


第二步是创建质量得分列表。
```
mawk '!/#/' DP3g95p5maf05.fil5.vcf | cut -f1,2,6 > DP3g95p5maf05.fil5.vcf.loci.qual
```

下一步是计算平均深度:
```
mawk '{ sum += $1; n++ } END { if (n > 0) print sum / n; }' DP3g95p5maf05.fil5.DEPTH
1952.82
```

现在平均值加上平均值的3倍
```
python -c "print int(1952+3*(1952**0.5))"
2084
```

接下来，我们将深度和质量文件粘贴在一起，找到截止点上方没有质量分数2倍深度的位点

```
paste DP3g95p5maf05.fil5.vcf.loci.qual DP3g95p5maf05.fil5.DEPTH | mawk -v x=2084 '$4 > x' | mawk '$3 < 2 * $4' > DP3g95p5maf05.fil5.lowQDloci
```
现在我们可以删除这些位点，并使用VCFtools重新计算基因座的深度。

```
cftools --vcf DP3g95p5maf05.fil5.vcf --site-depth --exclude-positions DP3g95p5maf05.fil5.lowQDloci --out DP3g95p5maf05.fil5
```

现在让我们将VCFtools输出并将其切割为仅深度分数
```
cut -f3 DP3g95p5maf05.fil5.ldepth > DP3g95p5maf05.fil5.site.depth
```

现在让我们通过将上述文件除以个体数量(31)来计算平均深度

```
mawk '!/D/' DP3g95p5maf05.fil5.site.depth | mawk -v x=31 '{print $1/x}' > meandepthpersite
```

具有高平均深度的基因座指示旁系同源物或多拷贝基因座。无论哪种方式，我们都想删除它们。在这里，我将删除平均深度为102.5以上的所有位点。现在我们可以组合两个过滤器来生成另一个VCF文件。

```
vcftools --vcf  DP3g95p5maf05.fil5.vcf --recode-INFO-all --out DP3g95p5maf05.FIL --max-meanDP 102.5 --exclude-positions DP3g95p5maf05.fil5.lowQDloci --recode 
```

最后，VCFtools在可能的9164位点中保留了8417个。

这个教程主要是展示了如果从多角度进行过滤的可能性，但实际上很多paper 或者project都不可能考虑到那么仔细，一般只是按照一些比较标准的过滤标准进行过滤（下节再讨论）。
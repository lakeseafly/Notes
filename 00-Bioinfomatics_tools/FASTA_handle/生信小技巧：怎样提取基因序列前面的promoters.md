# 生信小技巧：怎样提取基因序列前面的promoters

标签（空格分隔）： 生信小技巧

---

> 昨天隔壁实验室的老师让我帮忙提取目标基因的promoters，换句话说就是把目标序列前xxxxbp长度的基因序列提取出来。下面就和大家分享我解决这个问题的过程。

###问题解决思路

首先，我们要明确我们需要什么输入文件。我们的目标序列是基因序列前面的promoters，这部分信息当然是存放在**基因组序列的fasta**文件中（而不是蛋白质序列或者CDS序列中）。有了存储这些序列的基因组序列文件后，下一件事就是要找到该promoters的基因序列。那么怎么找呢？这时候我们需要先找到所对应基因在基因组序列的位置，然后再往前抓取xxxbp（该长度不同人有不同的定义），就是我们所需要的promoters的基因序列。那么这个基因的位置信息又是存储在哪里呢，简单当然是在**GFF基因注释**文件中。

总结一下输入文件就是：**基因组序列的fasta** 和 **GFF基因注释**

###解决问题实际操作

有了思路之后就要看你对整个生物信息学工具使用的熟练程度，不同人会有不同的方法去解决，有人会自己写代码写脚本达到目的，有人会通过综合运用现成的工具来解决问题。由于我代码技能不够溜，下面我就给大家提供**“通过综合运用现成的工具来解决问题”**的实操。当然这也只是其中一种解决方案，不一定是最快的方法。

**用到的工具**
都可以用过conda install去快速安装：

```
samtools
bedtools
bedops
```

**Step1**
首先先把gff文件中含有genes信息的行列抓取出来，并将其转化成bed格式，这里要用到bedops这个包中的gff2bed：

```
awk '/gbkey=Gene/' $var2 > genes.gff
gff2bed <genes.gff>genes.bed
```
**Step2**
将染色体或者contigs的长度计算出来：
```
samtools faidx Nt_example.fa

###这里会生成Nt_example.fa.fai,这里含有染色体或者contigs的长度，将这段信息提取出来

cut -f1-2 Nt_example.fa.fai >sizes.chr
```

**Step3**
根据你的genes.bed文件，创建含有promoters位置信息的bed文件：

```
bedtools flank -i genes.bed -g sizes.chr -l 3000 -r 0 -s > promoters.bed

### bedtools flank
###这个功能主要是用于创建一个含有位置信息的BED/GFF/VCF 文件
### -l The number of base pairs that a flank should start from orig. start coordinate.
### -r The number of base pairs that a flank should end from orig. end coordinate. 
###-s Define -l and -r based on strand.
###简单来说就是把-l 设置成你需要提取genes前xxxbp的promoters的长度

```

根据promoters的位置信息，在基因序列fasta文件中抓取promoters的序列：
```
bedtools getfasta -s -fi Nt_example.fa -bed promoters.bed -fo promoters.fa -name
```

这样就大功告成，整个过程简单和大家分享了。










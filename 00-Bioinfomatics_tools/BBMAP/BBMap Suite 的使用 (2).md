# 生信小工具专题：BBTools/BBMap Suite 的使用 (2)

标签（空格分隔）： 生信工具

---

> 接着上一次内容继续介绍，BBTools、BBMap中的一些实用的小工具。


###BBMap Read Merger
合并双末端（PE）reads,在预期中这些reads有重叠的位置（例如16S扩增子测序）。

通过重叠检测将配对的reads合并为单个read。如果具有足够的覆盖率，还可以通过kmer扩展合并非重叠的reads。 Kmer模式需要更多内存，应与bbmerge-auto.sh脚本一起使用。

```
Usage for interleaved files （交错的文件）:    bbmerge.sh in=<reads> out=<merged reads> outu=<unmerged reads>
Usage for paired files （有双末端的文件）:         bbmerge.sh in1=<read1> in2=<read2> out=<merged reads> outu1=<unmerged1> outu2=<unmerged2>
```


###BBMap bbsplit 


> 如何使用多个reference对我的样本中的reads进行多次比对？

这是一个很常见的问题，`bbsplit`是一个很好的工具去处理该问题。它非常灵活，可以处理多个数据序列。


同时比对多个基因组。此功能可用于将reads（可用于去污染）分类到“不同基因组”特定池中。

基本用法
```
To index:     bbsplit.sh build=<1> ref_x=<reference fasta> ref_y=<another reference fasta>
To map:       bbsplit.sh build=<1> in=<reads> out_x=<output file> out_y=<another output file>
```

然后这两个命令可以并在一起：

```
bbsplit.sh ref=x.fa,y.fa in=reads.fq basename=o%.fq

# this is equivalent to

bbsplit.sh build=1 in=reads.fq ref_x=x.fa ref_y=y.fa out_x=ox.fq out_y=oy.fq
```

默认情况下，配对读取将产生交错输出，但您可以使用＃符号生成双输出文件。例如，basename = o％_＃。fq将生成ox_1.fq，ox_2.fq，oy_1.fq和oy_2.fq。


我们现在将通过一个简单的例子来使用下面的bbsplit。

```
#工具下载
conda install entrez-direct

# 数据下载，获得来自NCBI的三种细菌基因组的fasta格式化序列。

efetch -db nucleotide -id NC_003454 -format fasta > NC_003454.fa
efetch -db nucleotide -id NC_010468 -format fasta > NC_010468.fa
efetch -db nucleotide -id NC_013520 -format fasta > NC_013520.fa

# 下载细菌的reads

wget http://data.biostarhandbook.com/reads/mixed-bacterial-reads.fq.gz

#
尽管我们在这用到的是细菌的reads，但这个方法通常适用于任何混合数据。我们可以使用一个或多个基因组的任意组合来将reads分类进每个对应的基因组中。

bbsplit.sh in=mixed-bacterial-reads.fq.gz ref=NC_010468.fa,NC_013520.fa,NC_003454.fa basename=out_%_#.fq.gz outu=rest.fq.gz minratio=0.76 minhits=1
```


bbsplit运行后应生成以下文件。

```
ls -sh1tr

2.4M rest.fq.gz
608K out_NC_013520_1.fq.gz
608K out_NC_003454_1.fq.gz
608K out_NC_010468_1.fq.gz
```

mixed-bacterial-reads.fq.gz文件现已被分成三个文件，其中包含我们感兴趣的reads。剩余的reads放在rest.fq.gz中


###BBMap repair 

配对末端（PE）序列是一种非常有用的工具，可提供有关被测序片段的空间信息。 PE的默认输出将读取对（指定为R1和R2）放在两个单独的文件中。当您扫描这些文件是否存在adapter序列和其他污染物时(相当于trimming，做质控的时候），建议将PE数据文件一起处理。 PE识别修剪/扫描程序使两个reads文件的reads顺序保一致。文件内部的读取顺序很重要，因为大多数NGS的mapping 工具都希望两个文件中的reads顺序相同（并且可能无法检查）。


如果文件被错误地trim或者某些文件被破坏，PE中的读取顺序可能会受到干扰。在最好的情况下，当ni使用此类文件作为NGS mapping 工具的输入时，这将产生错误。 （注意：在最坏的情况下，mapping可能会按原样处理数据。这可能会导致大量不一致的比对产生）。

使用BBMap工具中的`repair.sh`可以“重新同步保持一致”PE文件。这是一个举例说明的例子：

首先，人为先创造出一个不配对的fastq双末端的文件。


```
#下次会继续详细介绍这个工具，这里先使用
reformat.sh in=A1.2.fastq samplerate=0.9 out=A1.2.bad.fastq
```

这样子，就会产生一组不对称的双末端fastq file，一个是A1.1.fastq 另一个是A1.2.bad.fastq

接下来我们准备用`repair.sh`来修复'损坏的'reads配对并重新同步PE reads 以生成“固定”reads文件。 `repair.sh`的输出是交错的PE的reads（read1_R1，read1_R2，read2_R1，read2_R2等）。 “破碎”的reads将被存储于A1.bad.fastq中：

```
repair.sh in1=A1.1.fastq in2=A1.2.bad.fastq out=A1_fixed.fastq outsingle=A1.bad.fastq
```

我们现在可以通过`reformat.sh` 这个工具，将固定reads文件`A1_fixed.fastq`拆分为单独的R1 / R2 reads文件:

```
reformat.sh in=A1_fixed.fastq out1=A1.1.fixed.fastq out2=SRR1972739_2_fixed.fastq addcolon
```
简单来说，这个小工具是处理两个不一致的PE fastq文件的小神器。






# 生信小工具专题：BBTools/BBMap Suite 的使用 (3)

标签（空格分隔）： 生信工具

---

> 接着上一次内容继续给大家带来实用的BBTools的小教程。

###BBMap suite - Simulation of data （数据模拟）

除了之前介绍的工具，BBMap还可以用来，
从参考基因组产生随机合reads。reads的名称表示其基因组来源。并可以允许准确定的insert size和合成突变的类型，大小和速率等具体内容。 Illumina和PacBio类型的reads都可以使用`randomreads`这个小工具来生成。


您可以指定PE 的reads，insert size的分布，reads长度（或长度范围）等。 Randomreads专门设计用于对突变进行出色的控制。你可以准确地或概率地指定每个reads所包含的snps，insertion，deletions和Ns的数量;这些突变的长度可以进行单独的定制，质量值可以交替设置，以允许根据质量生成错误。所有reads都使用其基因组来源进行注释。

请记住，50％的reads将来自正链，50％来自负链。因此，reads将完全匹配参考基因组，或者它的反向补链将完美匹配。

基本用法

```
randomreads.sh ref=<file> out=<file> length=<number> reads=<number>
```
 
 介绍那么多，可能大家还是会有点疑惑究竟，这个工具有啥用？废话不多说，通过几行代码带你快速了解该工具的用处：
 
 

 1. 模拟高通量测序Illumina数据集。

首先先下载序列号为NC_010468 fasta格式的文件

```
efetch -db=nuccore -format=fasta -id=NC_010468 > NC_010468.fa
```
 
 
 
我们将通过以下命令简化NC_010468.fa中的fasta标头，以便新标头读取> NC_010468_E_coli

```
sed '1 s/^.*$/\>NC_010468_E_coli/' NC_010468.fa > NC_010468_new.fa
```


我们将使用此参考序列文件生成的合成illumina配对末端数据集（2 x 300 bp）
 
```
randomreads.sh ref=NC_010468_new.fa out=NC_010468_synth.fq.gz length=300 paired=t mininsert=100 maxinsert=400 reads=1000000
 
#默认情况下，生成的reads采用交错格式。它们可以通过`reformat.sh`进一步分成单独的read文件

reformat.sh in=NC_010468_synth.fq.gz out1=NC_010468_synth_R1.fq.gz out2=NC_010468_synth_R2.fq.gz addcolon
```
 
2. 使用randomreads生成PacBio格式数据集（使用PacBio错误模型）。

模拟一个包含10,000个读数的PacBio数据集，其最小读取长度为1kb，最大读取长度为5kb。

```
randomreads.sh ref=NC_010468_new.fa out=NC_010468_synth_pacbio.fq.gz minlength=1000 maxlength=5000 reads=10000 pacbio=t
```

3. 生成更复杂/有趣的合成数据集，可以模拟宏基因组。
下载数据
```
efetch -db=nuccore -format=fasta -id=NC_009614 > NC_009614.fa
efetch -db=nuccore -format=fasta -id=NC_013520 > NC_013520.fa
```

metagenome将为每个scaffold分配一个随机指数变量，该变量决定从该scaffold生成read的概率。因此，如果将20个细菌基因组连接在一起，则可以运行随机read并获得类似宏基因组的分布。当使用转录组参考时，它也可以用于RNA-seq。

覆盖率取决于每个参考序列水平，因此如果细菌组装具有多个contigs，您可能需要先将它们与fuse.sh粘在一起，然后再与其他参考物连接。

首先将ref 合成单一的文件：
```
cat NC_010468.fa NC_013520.fa NC_009614.fa AF086833.fa > metag.fa

```

然后我们将使用它作为宏基因组生成的输入，然后使用reformat.sh将读取分成两个文件。

```
randomreads.sh ref=metag.fa out=stdout length=300 paired=t mininsert=100 maxinsert=400 reads=5000000 metagenome=t coverage=3 | reformat.sh in=stdin out1=metag_R1.fq.gz out2=metag_R2.fq.gz interleaved addslash
```

### 重新格式化和过滤序列数据

Reformat.sh这个工具包前面一直有用到，接下来会详细讲解一下他的使用。

功能：
格式转换/操作/分析序列数据。

重新格式化reads以更改ASCII质量编码，交错，文件格式或压缩格式。可选择执行其他功能，如质量修整，子集和子采样。支持fastq，fasta，fasta，bam,gzip 等得格式。

基本用法:
```
reformat.sh in=<file> in2=<file2> out=<outfile> out2=<outfile2>
#in2 and out2 这两个options是PE reads的文件才会用到的（双末端）
```
具体应用
1. 切除一部分的reads文件

使用reformat.sh可以轻松地从文件中产生随机获取的reads。

```
#随机抽取300个reads
reformat.sh in=SRR1972739_1.fastq out=SRR1972739_1_sample.fastq sample=300


#您可以通过执行以总reads的百分比进行采样（以下示例中为10％）：
reformat.sh in=SRR1972739_1.fastq out=SRR1972739_1_10perc.fastq samplerate=0.1
```


2. 验证配对端数据的reads是正确配对的。
```
reformat.sh in1=r1.fq.gz in2=r2.fq.gz vpair
```



3. 按reads的长度过滤SAM / BAM文件
```
reformat.sh in=x.sam out=y.sam minlength=50 maxlength=200
```


4. 过滤/检测q SAM / BAM文件中的拼接reads


您可以将maxdellen（在下面的示例中为50）设置为你认为最小的任何长度的eletion事件，以表示拼接，这取决于你的物种的类型。

```
reformat.sh in=mapped.bam out=filtered.bam maxdellen=50
```

5. 
从SAM / BAM文件中获取比对不上的reads

```
reformat.sh in=all.sam out=unmapped.fq.gz unmappedonly
repair.sh in=unmapped.fq.gz  out=r1.fq.gz out2=r2.fq.gz outs=singleton.fq
```

###删除重复的序列数据集

从一组文件中删除重复的读取/序列。 类似picard tool的Markduplicates的功能。

```
dedupe.sh in=<file or stdin> out=<file or stdout>
```


在运行完成后，还会生成一组全面的统计信息。它们通常被写入\但可以很容易地捕获到日志文件中（例如下面的例子）

```
Input:                          15000 reads             1515000 bases.
Duplicates:                     7445 reads (49.63%)     751945 bases (49.63%)           0 collisions.
Containments:                   0 reads (0.00%)         0 bases (0.00%)         113772 collisions.
Result:                         7555 reads (50.37%)     763055 bases (50.37%)
```


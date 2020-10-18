#  生信小技巧：实用的one-liners命令(2)

标签（空格分隔）： 生信杂谈

---

> 接着上一次内容，继续和大家分享常用实用的one-liners命令。

###sort, uniq, cut, etc.

将文件中每一行的行号进行标识:

```
cat -n file.txt
```

对文件中独特的unique lines进行计数（每周都会用到不下五次以上）:
```
cat file.txt | sort -u | wc -l
```

对两个单行的文件进行sort，然后提取共有的元素（可以用来提取目标基因ID）:

```
sort -u file1 > a
sort -u file2 > b
comm -12 a b
```
对第几行的数字进行从小到大的排序:

```
sort -gk9 file.txt
```

随机抽打碎文件的顺序。并抽取1000行 (个人用过来随机抽取VCF文件中的变异位点，来画系统发育树）:
```
shuf file.txt | head -n 10
```


###find, xargs, and GNU parallel

在目前的directory中递归地搜索以.bam 结尾的文件：

```
find . -name "*.bam"
```

删除当前目录下所有以.bam结尾的文件(千万要小心使用):
```
find . -name "*.bam" | xargs rm
```



将所有以.txt结尾的文件重新命名为将.bak 结尾的文件（一般用来backup一些文件，当你想做一些大的处理或者改变的时候）:

```
find . -name "*.txt" | sed "s/\.txt$//" | xargs -i echo mv {}.txt {}.bak | sh
```

这里会提到parallel，一个很强并行运行文件的工具，下载链接：https://www.gnu.org/software/parallel/

同时并行运行12个fastqc的jobs:

```
find *.fq | parallel -j 12 "fastqc {} --outdir ."
```


并行index你的bam file文件，这里添加了一个(`--dry-run`）的option，表示只会把命令打印出来，并不会真正的执行，可以用来给我检查命令行有没有按照我们的要求输入对:

```
find *.bam | parallel --dry-run 'samtools index {}'
```

###seqtk

如果有了解过的同学，都会知道seqtk是一个很强大的Fasta/Fastq处理的一个工具,下载地址https://github.com/lh3/seqtk，或者直接通过`conda install seqtk`去安装。

将fastq格式的文件转化成fasta格式
```
seqtk seq -a in.fq.gz > out.fa
```
提取name.list文件中含有的序列名称的fa文件，在name.list(也可以输入bed文件）中一行有一个对应的header名称

```
seqtk subseq in.fa name.list > out.fa
```

从fastq文件中，随机抽取10000 read:
```
seqtk sample -s100 read1.fq 10000 > sub1.fq
seqtk sample -s100 read2.fq 10000 > sub2.fq
```
将头尾两端低质量的碱基切掉:
```
seqtk trimfq in.fq > out.fq
```

###对Gff文件的处理

Gff文件也是一个我们日常经常遇到要处理的文件

找出你成功被注释的序列名:
```
cut -s -f 1,9 yourannots.gff3 | grep $'\t' | cut -f 1 | sort | uniq
```

找出你gff文件中注释的类型(exon/gene/mRNAd/CDS等）

```
grep -v '^#' yourannots.gff3 | cut -s -f 3 | sort | uniq
```

统计GFF文件中，能注释到gene的数目:
```
grep -c $'\tgene\t' yourannots.gff3
```

提取GFF文件中的gene ID:
```
grep $'\tgene\t' yourannots.gff3 | perl -ne '/ID=([^;]+)/ and printf("%s\n", $1)'
```



计算gff文件中，基因的长度：

```
grep $'\tgene\t' yourannots.gff3 | cut -s -f 4,5 | perl -ne '@v = split(/\t/); printf("%d\n", $v[1] - $v[0] + 1)'
```


----------
这里我就选了一些个人觉得比较熟悉的命令来分享，欢迎大家在留言区补充其它常用的命令。





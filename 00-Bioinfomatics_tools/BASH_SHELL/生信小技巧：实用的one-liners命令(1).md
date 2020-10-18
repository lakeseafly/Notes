# 生信小技巧：实用的one-liners命令(1)

标签（空格分隔）： 生信小技巧

---

> 当你处理NGS文本信息时，相信你不多不少也有遇到过类似的烦恼，例如不知道如何只提取gene ID的名字，或者只提取gff file中某一列的某一行，又或者想自己在每一行中添加一些你想要的信息。这时候如果你会编程，写一个脚本，这都不是什么问题。当然如果你编程不是很过关，掌握一些实用的one-liners命令，往往会事半功倍，帮助你实现一些简单的文本操作。

本文给大家分享一些实用的one-liners命令，如果觉得实用欢迎收藏与转发。

###基本的awk命令

提取一个文本中第2,4,和5行。
```
awk '{print $2,$4,$5}' input.txt
#对比cut，awk会相对更加好用。
```

提取第五列中等于0的所有行：

```
awk '$5 == "0"' file.txt
```
提取第五列中不等于0的所有行：

```
awk '$5 ！= "0"' file.txt
```

根据第二列的字符串来提取unique的文本内容，即去重（只提取第一个unique字符串的那一行）

```
awk '!arr[$2]++' file.txt
#类似sort -u，不过 可以精准到你想以那一行的字符串来进行去重复
```

提取出第三列大于第五列的行数：
```
awk '$3>$5' file.txt

##当然也可以提取出大于每个值得行，可以用于提取大于某个长度的gene的那一行。
awk '$3> 500' file.txt
```

求第一列的和：
```
awk '{sum+=$1} END {print sum}' file.txt
#可用于计算基因长度的总和等
```

求第二列数值的平均值，

```
awk '{x+=$2}END{print x/NR}' file.txt
#可以用来求基因长度平均值
```

###基本的sed命令

把文件中所有的`foo`
替代成`bar`

```
sed 's/foo/bar/g' file.txt
```

去除文件中开头的空格或者tab字符：

```
sed 's/^[ \t]*//' file.txt
```
去除文件中结尾的空格或者tab字符：

```
sed 's/[ \t]*$//' file.txt
```

同时去除文件开头与结尾的空格或者tab字符：

```
sed 's/^[ \t]*//;s/[ \t]*$//' file.txt
```

删除文本中的空白行

```
sed '/^$/d' file.txt
#有些工具不接受有空白行的输入文件
```

删除文件中行中包含abc的行及其这行以后的所有行：
```
sed -n '/abc/,$!p' file.txt
```

###生物信息中的awk和sed

提取file.txt中 Chr1 中1MB和2MB的片段之间的行信息，假设第一列是Chr信息，第三列是位置信息：

```
cat file.txt | awk '$1=="1"' | awk '$3>=1000000' | awk '$3<=2000000'
```

统计fastq文件中基本的参数，包括总的reads数目，总的unique reads的数目(就是不重复的reads），这些unique reads的比例，最过剩的read的序列及其数目与占总的reads的比率：

```
at myfile.fq | awk '((NR-2)%4==0){read=$1;total++;count[read]++}END{for(read in count){if(!max||count[read]>max) {max=count[read];maxRead=read};if(count[read]==1){unique++}};print total,unique,unique*100/total,maxRead,count[maxRead],count[maxRead]*100/total}'
#拿到raw data来统计时候很实用
```

将bam格式的文件转化为fastq格式：

```
samtools view file.bam | awk 'BEGIN {FS="\t"} {print "@" $1 "\n" $10 "\n+\n" $11}' > file.fq
```

将一个multi-FASTA 分成多个的单个FASTA文件:
```
awk '/^>/{s=++d".fa"} {print > s}' multi.fa
```

将fastq文件转变成fa文件:

```
sed -n '1~4s/^@/>/p;2~4p' file.fq > file.fa
```
输出fasta文件中每一个序列的header名字与其的长度

```
cat file.fa | awk '$0 ~ ">" {print c; c=0;printf substr($0,2,100) "\t"; } $0 !~ ">" {c+=length($0);} END { print c; }'
```
输出制定的行数:

```
#例如20到80行
awk 'NR>=20&&NR<=80' input.txt
```

转化 VCF file 成BED file:

```
sed -e 's/chr//' file.vcf | awk '{OFS="\t"; if (!/^#/){print $1,$2-1,$2,$4"/"$5,"+"}}'
```

计算vcf 中 每一个line的missing样本数目：

```
bcftools query -f '[%GT\t]\n' a.bcf |  awk '{miss=0};{for (x=1; x<=NF; x++) if ($x=="./.") {miss+=1}};{print miss}' > nmiss.count
```

根据reads的名字，提取fastq中的reads:
```
zcat a.fastq.gz | awk 'BEGIN{RS="@";FS="\n"}; $1~/readsName/{print $2; exit}'
```
计算fastq文件中reads 长度的平均值:

```
awk 'NR%4==2{sum+=length($0)}END{print sum/(NR/4)}' input.fastq
```

本次主要讲sed 和 awk 两大神器，下周会更新其他实用的one-liners command，希望对大家有帮助，如果大家也有其他实用的one-liners command也欢迎大家在回复栏中分享你收藏的command。


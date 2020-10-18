# # Super useful!! #实用的单行命令合集

标签（空格分隔）： 生信小技巧

---

周末闲着无聊在推特上闲逛，刷着刷着发现了Ming Tang大神的一条推文。他将他日常经常使用的单行命令整理到其Github中，然后分享给各位推友。在点开它的网站后，我犹如发现了一堆宝藏。里面很多单行命令行非常实用，快速简单地解决了我日常所遇到的生物信息学数据处理问题(没有这些单行命令，我往往要通过自己写脚本或者借用其它工具来解决日常的生物信息学数据处理)。在取得Ming Tang大神的同意后，该推文将和大家整理分享这一系列实用的单行命令。

原文链接：
**GitHub** 地址： https://github.com/stephenturner/oneliners


### Fastq/Fasta文件的处理

统计fastq文件中的read的长度与不同长度的分布：

```
zcat file.fastq.gz | awk 'NR%4 == 2 {lengths[length($0)]++} END {for (l in lengths) {print l, lengths[l]}}'  
```

fastq转换为fasta格式:

```
zcat file.fastq.gz | paste - - - - | perl -ane 'print ">$F[0]\n$F[2]\n";' | gzip -c > file.fasta.gz
```

统计fastq文件中reads的数目:
```
cat file.fq | echo $((`wc -l`/4))
```

随机抽取fastq文件的一部分，下面使用提取0.01%的reads作为例子(在不知道该方法之前，我一般使用Heng Li的seqtk sample来达到该目的):
```
cat file.fq | paste - - - - | awk 'BEGIN{srand(1234)}{if(rand() < 0.01) print $0}' | tr '\t' '\n' > out.fq
```

从fastq文件中提取含有某个碱基排列序列的read，并且让其保持fastq的格式(含有四行的信息:

```
### 方法一
grep -A 2 -B 1 'AAGTTGATAACGGACTAGCCTTATTTT' file.fq | sed '/^--$/d' > out.fq

### 方法二
zcat reads.fq.gz \
| paste - - - - \
| awk -v FS="\t" -v OFS="\n" '$2 ~ "AAGTTGATAACGGACTAGCCTTATTTT" {print $1, $2, $3, $4}' \
| gzip > filtered.fq.gz
```

分选fastq文件，有些fastq文件是同时包含了R1,R2的序列信息，需要将它们分离出来:

```
cat file.fq | paste - - - - - - - - | tee >(cut -f1-4 | tr '\t'  
'\n' > out1.fq) | cut -f5-8 | tr '\t' '\n' > out2.fq
```

将一个含有多个染色体或者contigs的fasta文件，按照染色体或者contigs分离成多个只含单一序列的fasta文件:

```
awk '/^>/{s=++d".fa"} {print > s}' multi.fa
```

将fasta文件中多行的序列变成单行:

```
### 方法一
cat file.fasta | awk '/^>/{if(N>0) printf("\n"); ++N; printf("%s\t",$0);next;} {printf("%s",$0);}END{printf("\n");}'

### 方法二
awk 'BEGIN{RS=">"}NR>1{sub("\n","\t"); gsub("\n",""); print RS$0}' file.fa
```

反过来将单行的fasta文件，变成多行，下面例子对每60个碱基分一行:

```
awk -v FS= '/^>/{print;next}{for (i=0;i<=NF/60;i++) {for (j=1;j<=60;j++) printf "%s", $(i*60 +j); print ""}}' file

fold -w 60 file
```

统计mulitfasta中，每个染色体/contigs的长度:

```
awk '/^>/ {if (seqlen){print seqlen}; print ;seqlen=0;next; } { seqlen = seqlen +length($0)}END{print seqlen}' file.fa
```

对fasta文件中的header中添加对应的信息:

```
### 目前你有的fasta文件
cat myfasta.txt 
>Blap_contig79
MSTDVDAKTRSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
>Bluc_contig23663
MSTNVDAKARSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
>Blap_contig7988
MSTDVDAKTRSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
>Bluc_contig1223663
MSTNVDAKARSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI

###需要添加的信息
cat my_info.txt 
info1
info2
info3
info4

### 添加信息到对应的fasta header中
paste <(cat my_info.txt) <(cat myfasta.txt| paste - - | cut -c2-) | awk '{printf(">%s_%s\n%s\n",$1,$2,$3);}'
>info1_Blap_contig79
MSTDVDAKTRSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
>info2_Bluc_contig23663
MSTNVDAKARSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
>info3_Blap_contig7988
MSTDVDAKTRSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
>info4_Bluc_contig1223663
MSTNVDAKARSKERASIAAFYVGRNIFVTGGTGFLGKVLIEKLLRSCPDVGEIFILMRPKAGLSI
```

简化fasta文件的表头,比如将`>7 dna:chromosome chromosome:GRCh37:7:1:159138663:1`转化为`>7`
```
cat Homo_sapiens_assembly19.fasta | gawk '/^>/ { b=gensub(" dna:.+", "", "g", $0); print b; next} {print}' > Homo_sapiens_assembly19_reheader.fasta
```

### BAM/VCF/BED文件的处理

将BAM文件转化成bed格式
```
samtools view file.bam | perl -F'\t' -ane '$strand=($F[1]&16)?"-":"+";$length=1;$tmp=$F[5];$tmp =~ s/(\d+)[MD]/$length+=$1/eg;print "$F[2]\t$F[3]\t".($F[3]+$length)."\t$F[0]\t0\t$strand\n";' > file.bed
```

将BAM文件转化成wig格式

```
samtools mpileup -BQ0 file.sorted.bam | perl -pe '($c, $start, undef, $depth) = split;if ($c ne $lastC || $start != $lastStart+1) {print "fixedStep chrom=$c start=$start step=1 span=1\n";}$_ = $depth."\n";($lastC, $lastStart) = ($c, $start);' | gzip -c > file.wig.gz
```

由于samtools mpileup只能使用1个CPU，按照染色体，并使用xargs去并行call variant能够极大地提高其变异发现的效率:

```
samtools view -H yourFile.bam | grep "\@SQ" | sed 's/^.*SN://g' | cut -f 1 | xargs -I {} -n 1 -P 24 sh -c "samtools mpileup -BQ0 -d 100000 -uf yourGenome.fa -r {} yourFile.bam | bcftools view -vcg - > tmp.{}.vcf"
```
另外在分染色体call完变异后，也可以将不同染色体的变异合并起来:

```
samtools view -H yourFile.bam | grep "\@SQ" | sed 's/^.*SN://g' | cut -f 1 | perl -ane 'system("cat tmp.$F[0].bcf >> yourFile.vcf");'
```

寻找当前目录下的bam文件，并使用5个CPUs将其复制到其它新的目录中(这个命令在转移文件时，非常实用):

```
find . -name "*bam" | xargs -P5 -I{} rsync -av {} dest_dir
```

根据vcf的表头文件来sort VCF文件

```
cat my.vcf | awk '$0~"^#" { print $0; next } { print $0 | "sort -k1,1V -k2,2n" }'
```

将含有PASS的变异从VCF中提取出来(经过一些工具过滤后，例如GATK，如果符合过滤的阀值，在VCF文件中会有一行PASS的字符):

```
cat my.vcf | awk -F '\t' '{if($0 ~ /\#/) print; else if($7 == "PASS") print}' > my_PASS.vcf
```
将bed文件按照不同的染色体分开(同样的道理可以应用到gff文件中):

```
### 方法一
cat nexterarapidcapture_exome_targetedregions_v1.2.bed | sort -k1,1 -k2,2n | sed 's/^chr//' | awk '{close(f);f=$1}{print > f".bed"}'

### 方法二
awk '{print $0 >> $1".bed"}' example.bed
```

### 处理其它文件/序列

将序列转化其互补的序列，在设计primers很有用:

```
echo 'ATTGCTATGCTNNNT' | rev | tr 'ACTG' 'TGAC'
```

重命名一系列文件:

```
for file in *gz
do zcat $file > ${file/bed.gz/bed}
```

将看不到的字符显示出来
```
### 方法一
cat my_file | sed -n 'l'
### 方法二
cat -A
```

复制大量的文件，使用rsync比copy速度更快
```
## 将所有文件从一个目录复制到另一个目录
rsync -av from_dir/ to_dir
##将所有文件从一个目录复制到另一个目录，但是已经复制好的文件将会被跳过:
rsync -avhP /from/dir /to/dir
```
显示当前目录下所有的子目录所占空间的大小
```
du -h --max-depth=1
```

查看当前内存的使用情况
```
free -mg
```


根据第一和第二列打印出特殊唯一的行:
```
awk '!a[$1,$2]++' input_file
### 或者
sort -u -k1,2 file
```

删除空白行:

```
sed /^$/d
```
删除最后一行
```
sed $d
```

使用`!$`,使用最后一个命令的字符

```
echo hello, world; echo !$
```
将最后执行的命令保存为脚本:

```
echo "!!" > foo.sh
```
重复使用刚刚执行命令的参数:
```
!*
```

统计一个tsv文件中有多少列:

```
cat file.tsv | head -1 | tr "\t" "\n" | wc -l  

##csvkit工具中的csvcut
csvcut -n -t file.

## 另一些可行的方法
less files.tsv | head -1| tr "\t" "\n" | nl
awk -F "\t" 'NR == 1 {print NF}' file.tsv
awk '{print NF; exit}'
```

创建一个新的文件夹，并且马上`cd`进去
```
mkdir blah && cd $_
```

切割所需要的行，根据另一个文件提供的行名称(Thanks God,我一直想找现成的办法去处理这个问题,非常合适处理大型的文件，比R占更少的内存):

```
#! /bin/bash

set -e
set -u
set -o pipefail

#### Author: Ming Tang (Tommy)
#### Date 09/29/2016
#### I got the idea from this stackOverflow post http://stackoverflow.com/questions/11098189/awk-extract-columns-from-file-based-on-header-selected-from-2nd-file

# show help
show_help(){
cat << EOF
  This is a wrapper extracting columns of a (big) dataframe based on a list of column names in another
  file. The column names must be one per line. The output will be stdout. For small files < 2G, one 
  can load it into R and do it easily, but when the file is big > 10G. R is quite cubersome. 
  Using unix commands on the other hand is better because files do not have to be loaded into memory at once.
  e.g. subset a 26G size file for 700 columns takes around 30 mins. Memory footage is very low ~4MB.

  usage: ${0##*/} -f < a dataframe  > -c < colNames> -d <delimiter of the file>
        -h display this help and exit.
		-f the file you want to extract columns from. must contain a header with column names.
		-c a file with the one column name per line.
		-d delimiter of the dataframe: , or \t. default is tab.  
		
		e.g. 
		
		for tsv file:
			${0##*/} -f mydata.tsv -c colnames.txt -d $'\t' or simply ommit the -d, default is tab.
		
		for csv file: Note you have to specify -d , if your file is csv, otherwise all columns will be cut out.
			${0##*/} -f mydata.csv -c colnames.txt -d ,
        
EOF
}

## if there are no arguments provided, show help
if [[ $# == 0 ]]; then show_help; exit 1; fi

while getopts ":hf:c:d:" opt; do
  case "$opt" in
    h) show_help;exit 0;;
    f) File2extract=$OPTARG;;
    c) colNames=$OPTARG;;
    d) delim=$OPTARG;;
    '?') echo "Invalid option $OPTARG"; show_help >&2; exit 1;;
  esac
done
	

## set up the default delimiter to be tab, Note the way I specify tab 

delim=${delim:-$'\t'}

## get the number of columns in the data frame that match the column names in the colNames file.
## change the output to 2,5,6,22,... and get rid of the last comma  so cut -f can be used
 
cols=$(head -1 "${File2extract}" | tr "${delim}" "\n" | grep -nf "${colNames}" | sed 's/:.*$//' | tr "\n" "," | sed 's/,$//')

## cut out the columns 
cut -d"${delim}" -f"${cols}" "${File2extract}"
```
相同的问题，使用`csvtk`就简单很多了:

```
csvtk cut -t -f $(paste -s -d , list.txt) data.tsv
```

在每行开始的地方增加或者删除chr，有些软件不支持chr字符，需要使用数字来表示染色体:

```
# 增加 chr
sed 's/^/chr/' my.bed

# 或者
awk 'BEGIN {OFS = "\t"} {$1="chr"$1; print}'

# 移除 chr
sed 's/^chr//' my.bed
```

检查 tsv文件中行与列的数目的是否一致:
```
awk '{print NF}' test.tsv | sort -nu | head -n 1
```

只替代其中一列中的某个pattern
```
## 替代第五列
awk '{gsub(pattern,replace,$5)}1' in.file

## 使用强大的csvtk
csvtk replace -f 5 -p pattern -r replacement
```

反转文件中的其中一列

```
###反转第三列然后将其结果放到第五列中
awk -v OFS="\t" '{"echo "$3 "| rev" | getline $5}{print $0}' 
###只反转第二列
perl -lane 'BEGIN{$,="\t"}{$rev=reverse $F[2];print $F[0],$F[1],$rev,$F[3]}
```

从gtf文件中提取promoter的位置，例子提取基因上下游5kbp的序列

```
zcat gencode.v29lift37.annotation.gtf.gz | awk '$3=="gene" {print $0}' | grep protein_coding | awk -v OFS="\t" '{if ($7=="+") {print $1, $4, $4+5000} else {print $1, $5-5000, $5}}' > promoters.bed
```

但是值得注意的是，某些基因位于染色体的末端，提取上下游5kbp可能超出了这一点，这种情况下最好请使用`bedtools slop`根据基因组大小文件来避免这种情况。

```
## 使用fetchChromSizes提取染色体大小，下载地址http://hgdownload.soe.ucsc.edu/admin/exe/linux.x86_64/
fetchChromSizes hg19 > chrom_size.txt
### 提取对应的promoter区域
zcat gencode.v29lift37.annotation.gtf.gz | awk '$3=="gene" {print $0}' |  awk -v OFS="\t" '{if ($7=="+") {print $1, $4, $4+1} else {print $1, $5-1, $5}}' | bedtools slop -i - -g chrom_size.txt -b 5000 > promoter_5kb.bed
```

好啦，干货满满的介绍到此就结束了，希望大家有所收获。如果觉得该文章有帮助到你，欢迎转发，点好看，分享给更多的生信小伙伴。最后，再次感谢Ming Tang大神提供的资源。
# 生信小白必备工具：BLAST

标签（空格分隔）： 生信工具

---

这期开始将会系统的介绍一些生信初学者经常使用的工具，流程，敬请大家关注。首先今天和大家讲讲本地版的BLAST，一款作为生信工作者的你，这个是你每天都会用到的工具。这个网页版的blast估计你一定不会陌生吧？
![][1]


###基本介绍

BLAST是Basic Local Alignment Search Tool的缩写，直接翻译过来就是**基本局部比对搜索工具**。

常用的BLAST program有四个：

| 查询序列类型 | 数据库类型 | BLAST program |
| ------ | ------ | ------ |
| 核苷酸 | 核苷酸 | blastn: 使用核苷酸序列去搜索核苷酸数据库 |
| 核苷酸 | 蛋白质 | blastx: 使用核苷酸序列去搜索蛋白质数据库 |
| 蛋白质 | 核苷酸 | tblastn: 使用蛋白质序列搜索核苷酸数据库 |
| 蛋白质 | 蛋白质 | blastp：使用蛋白质序列搜索蛋白质数据库 |


###实战如何使用blast寻找同源基因

科学家在鱼（Seriola quinqueradiata）中发现了一种与寄生虫（Benedenia）抗性相关的基因（C-凝集素）。我们实战的问题中的目标是在琥珀鱼(Seriola rivoliana)基因组中找到相应的基因。

![][2]


首先，从NCBI下载Seriola rivoliana基因组

```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/002/994/505/GCA_002994505.1_ASM299450v1/GCA_002994505.1_ASM299450v1_genomic.fna.gz

#unzip
gunzip GCA_002994505.1_ASM299450v1_genomic.fna.gz
```

获得C-凝集素蛋白质序列

```
vi benediniaGene.fasta

###将下面序列

>C-LECTIN
MKTLLILSVVLCAALSVRAAAVVPAEAATAQLGDKAAPEPEAVKDTAVEDTAVEETAVEDTAVEETAVEDTAVEETAVEDTAVEETAVEDTAVEDTAVEDTAVEDTAVEDTAVEETAVEDTAVEDTAVEDTAVAAGRPAGLRQTRLSFCLDGWQSFSGKCYFLANHPDSWANAERFCASYEGSLASVGSIWEYNFLQRMVKTGGHAFAWIGGYYFQGEWRWEDGSRFDY
SNWDTPRSTAYYQCLLLNSQVSMGWSNNGCNMNFPFVCQVRQLNC

```

创建基因组的核苷酸序列数据库


```
makeblastdb -in GCA_002994505.1_ASM299450v1_genomic.fna -dbtype nucl -input_type fasta -out SerRivdb
```


参数:

 - -in：在这里，我们使用S. quinqueradiata基因组。
 - -dbtype：我们希望生成的数据库类型，这里我们输入是核苷酸序列，选用nucl。如果是蛋白质序列作为输入，可以使用prot
-input_type：输入文件的类型（fasta）
-out：要生成的数据库的输出前缀

输出结果看起来像这样:

```
Building a new DB, current time: 08/03/2018 15:32:36
New DB name:   /pylon5/mc48o5p/severin/Seriola/Rivoliana/SerRivdb
New DB title:  GCA_002994505.1_ASM299450v1_genomic.fna
Sequence type: Nucleotide
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 1343 sequences in 5.96571 seconds.
```

建库完成之后，我们要使用蛋白质序列对核苷酸数据库进行搜索，因此我们需要BLAST第中tblastn的program，其基本用法如下:

```
tblastn -db SerRivdb -query benedeniaGene.fasta  -out blastout.txt
```
参数

 - -db：由makeblastdb和参考fasta文件生成的基因组参考数据库
 - -query：我们这里是要比对的蛋白质序列文件
 - -out：你希望将结果输出的文件

默认OUTPUT结果

```
> PVUN01001342.1 Seriola rivoliana isolate HWSR04 Scaffold_1308,
whole genome shotgun sequence
Length=25273714

 Score = 72.4 bits (176),  Expect(2) = 3e-21, Method: Compositional matrix adjust.
 Identities = 32/36 (89%), Positives = 33/36 (92%), Gaps = 0/36 (0%)
 Frame = -3

Query  214      YFQGEWRWEDGSRFDYSNWDTPRSTAYYQCLLLNSQ  249
                + Q EWRWEDGSRFDYSNWDTP STAYYQCLLLNSQ
Sbjct  2542610  FLQDEWRWEDGSRFDYSNWDTPSSTAYYQCLLLNSQ  2542503


 Score = 51.2 bits (121),  Expect(2) = 3e-21, Method: Compositional matrix adjust.
 Identities = 22/25 (88%), Positives = 23/25 (92%), Gaps = 0/25 (0%)
 Frame = -2

Query  250      VSMGWSNNGCNMNFPFVCQVRQLNC  274
                VS GWSNNGCNM FPFVCQVRQL+C
Sbjct  2542419  VSKGWSNNGCNMRFPFVCQVRQLDC  2542345


 Score = 85.5 bits (210),  Expect = 3e-17, Method: Compositional matrix adjust.
 Identities = 39/54 (72%), Positives = 44/54 (81%), Gaps = 0/54 (0%)
 Frame = -3

Query  163      LANHPDSWANAERFCASYEGSLASVGSIWEYNFLQRMVKTGGHAFAWIGGYYFQ  216
                + N   S    +RFCAS++GSLASV SIWEYNFLQRMVKTGGH FAWIGGYYF+
Sbjct  2543138  IKNKSSSPVVLQRFCASFDGSLASVRSIWEYNFLQRMVKTGGHKFAWIGGYYFE  2542977
```
为了输出更加简洁，这里调整一下输出的格式：

```
tblastn -db SerRivdb -query benediniaGene.fasta  -outfmt '6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore qcovs salltitles' -num_threads 16 -out blastout2.txt
```

这样每行每列都会按照你输出的格式来输出，blast结果有很多内容，但是我们一般重点关注e-`value`，比对上的长度 `length` 还有比对的相似度 `pident`。这里我们查看一下具有相似度50%以上的blast结果：

```
more blastout2.txt  | awk '$3>50'
C-LECTIN        PVUN01001342.1  88.889  36      4       0       214     249     2542610 2542503 3.15e-21        72.4    49      PVUN01001342.1 Seriola rivoliana isolate HWSR04 Scaffold_1308, whole genome shotgun sequence
C-LECTIN        PVUN01001342.1  88.000  25      3       0       250     274     2542419 2542345 3.15e-21        51.2    49      PVUN01001342.1 Seriola rivoliana isolate HWSR04 Scaffold_1308, whole genome shotgun sequence
C-LECTIN        PVUN01001342.1  72.222  54      15      0       163     216     2543138 2542977 3.04e-17        85.5    49      PVUN01001342.1 Seriola rivoliana isolate HWSR04 Scaffold_1308, whole genome shotgun sequence
C-LECTIN        PVUN01001342.1    85.714  35      5       0       140     174     2543329 2543225 4.08e-11        67.0    49      PVUN01001342.1 Seriola rivoliana isolate HWSR04 Scaffold_1308, whole genome shotgun sequence
```

这样子你就可以找到C-LECTIN蛋白在琥珀鱼(Seriola rivoliana)基因组中同源相似的序列（在PVUN01001342.1上2542610bp位置附近)。然后你就可以基于这一段序列，做一些列下游的分析实验验证了。好了介绍差不多了，当然这里没有完全介绍完blast program的所有参数，这个大家可以去其帮助文档或者手册进行进一步的学习。对运行大批量的蛋白质或者核苷酸序列进行本地nt数据库的搜索，推荐大家可以使用将大块的序列切成小块，再通过并行运行来缩短运行的时间。








  [1]: http://static.zybuluo.com/lakesea/d5naojrd4m8h7nn9u1accl9q/Screen%20Shot%202019-05-08%20at%207.52.49%20pm.png
  [2]: http://safmc.net/wp-content/uploads/2016/06/almaco_jack.jpg
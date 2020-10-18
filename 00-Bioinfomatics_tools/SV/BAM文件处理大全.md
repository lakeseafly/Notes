# SAM/BAM文件处理大全

标签（空格分隔）： 生信技术

---

最近一直在处理一些BAM文件，由于对某些命令不够熟悉，处理效率不太高。于是今天借此机会，给大家总结一下SAM/BAM文件的处理命令。希望对大家日后处理SAM/BAM起到帮助。

### samtools的安装

在讲到如何具体安装之前，说说samtools的版本问题。samtools不同版本之间是有差异的，特别是比较旧的1.3一下的版本，与新版本之前有较大的不同。最近听说samtools又推出了最新的版本1.10 （不太懂版本的名字是如何设置的），功能速度又有了进一步的提高。

 1. 安装途径有很多，最简单的就是通过`conda`安装了，就不具体说了。
 2. 或者也可以通过手动安装,这里我没安装最新的版本，使用了我之前已经下载好的较新的1.7版本:
```
wget https://github.com/samtools/htslib/releases/download/1.7/htslib-1.7.tar.bz2
tar -xjf htslib-1.7.tar.bz2
cd htslib-1.7
make
make prefix=. install
cd ..

wget https://github.com/samtools/samtools/releases/download/1.7/samtools-1.7.tar.bz2
tar -xjf samtools-1.7.tar.bz2
cd samtools-1.7
make
make prefix=. install
cd ..
```
  
### samtools 总概

作为最强大的工具之一，samtools里面含有很多子功能，从文件处理，数据统计，文件的编辑等一系列的功能。
```
samtools

Program: samtools (Tools for alignments in the SAM format)
Version: 1.7 (using htslib 1.7)

Usage:   samtools <command> [options]

Commands:
  -- Indexing
     dict           create a sequence dictionary file
     faidx          index/extract FASTA
     index          index alignment

  -- Editing
     calmd          recalculate MD/NM tags and '=' bases
     fixmate        fix mate information
     reheader       replace BAM header
     targetcut      cut fosmid regions (for fosmid pool only)
     addreplacerg   adds or replaces RG tags
     markdup        mark duplicates

  -- File operations
     collate        shuffle and group alignments by name
     cat            concatenate BAMs
     merge          merge sorted alignments
     mpileup        multi-way pileup
     sort           sort alignment file
     split          splits a file by read group
     quickcheck     quickly check if SAM/BAM/CRAM file appears intact
     fastq          converts a BAM to a FASTQ
     fasta          converts a BAM to a FASTA

  -- Statistics
     bedcov         read depth per BED region
     depth          compute the depth
     flagstat       simple stats
     idxstats       BAM index stats
     phase          phase heterozygotes
     stats          generate stats (former bamcheck)

  -- Viewing
     flags          explain BAM flags
     tview          text alignment viewer
     view           SAM<->BAM<->CRAM conversion
     depad          convert padded BAM to unpadded BAM
```

由于其功能太多，我下面会挑选一部分经常用到的和大家分享:


### 使用samtools view 查看
该功能应该是最基本的功能，查看比对生成的bam/sam格式的结果。

```
samtools view aln.sam | less
```

![][1]

### 将SAM格式转化为BAM格式

BAM文件的内容和SAM文件，但是以二进制形式存储；我们一般都会将SAM文件转换为BAM，以节省存储空间，并且BAM文件的处理速度更快。

首先，我们简单回顾一下SAM文件的格式:

```
head aln.sam 
@SQ     SN:1000000      LN:1000000
@PG     ID:bwa  PN:bwa  VN:0.7.13-r1126 CL:bwa/bwa mem sequence/ref.fa sequence/l100_n10000_d300_31_1.fq sequence/l100_n10000_d300_31_2.fq
1:165617        99      1000000 165617  60      100M    =       165917  400     TGCAGTGGTATCGGATCAGCCTAGATGCCATAGCTGAGCGCCAAATTTCCGGATTTTCCCCGTGTAGTCAATGGAGCTGTTACTTTAAGCCGTGAATGTG    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ    NM:i:0  MD:Z:100        AS:i:100        XS:i:0
1:165617        147     1000000 165917  60      100M    =       165617  -400    AATTCGATATGCCGGTCATCGTGTGTCTATGATACTCCTTAGGCATCCCTTTAACTACGATACTTTAAGAGGTGCGAAAAGTATTCTATACGGCAGCGTA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ    NM:i:0  MD:Z:100        AS:i:100        XS:i:0
2:591911        99      1000000 591911  60      100M    =       592211  400     GATCAAGCTGGGCATGGGTTCGGTGACGCGTAAAAAAATTTTTTTCTGAGGACCACTGAGAAGATGGTTACGTCTAGGATCTAAGACCTAGTGTCAACTC    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ    NM:i:0  MD:Z:100        AS:i:100        XS:i:0
2:591911        147     1000000 592211  60      100M    =       591911  -400    GCATGACACTGGATAGTGCGATTAGATAGCGGCTCGGGAGTACGTCACTGAAAGTCCAGTGCGAGAGCCAACCCGGAAACTCTACATGCGCATGTAGAAC    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ    NM:i:0  MD:Z:100        AS:i:100        XS:i:0
```

以**@**符号开头的行,包含表头信息。**@SQ**标签是参考序列字典；**SN**表示参考序列名称，**LN**表示参考序列长度。如果看不到以**@**开头的行，那么你的SAM文件可能就是缺少了表头的信息。如果SAM文件中没有标表头，可以使用以下命令进行转化，其中**ref.fa**是用比对的参考序列文件：

```
samtools view -bT sequence/ref.fa aln.sam > aln.bam
```

如果你的文件有表头:

```
samtools view -bS aln.sam > aln.bam

###在新版的samtools中可以不需要—S这个参数，因为新版本的工具可以自动探索输入文件的格式。
samtools view -b aln.sam > aln.bam
```

### 将BAM继续转化为CRAM格式

CRAM格式可以进一步压缩BAM的存储空间,比如`SAM = 5.5M, unordered BAM = 978K, ordered BAM = 746K, and CRAM = 91K`:

```
samtools view -T sequence/ref.fa -C -o aln.cram aln.bam
```

### Sort BAM 文件
通常来说sort BAM文件是一个很好的习惯，因为大部分下游的分析软件都需要sorted好的BAM文件
```
samtools sort aln.bam -o aln.bam
```

### 一步到位直接从SAM文件到sorted好的BAM文件

一步到位就很舒服啦：
```
samtools view -bS aln.sam | samtools sort - -o aln.bam
```

在1.3版本或者更新的版本，可以直接sort SAM文件:

```
samtools sort -o aln.bam aln.sam
```
当然也可以使用更多的线程进行加速这个过程：
```
samtools sort -@ 8 -o aln.bam aln.sam
```
### 转化BAM回到SAM文件

如果你想反过来转换格式也是可以的:
```
##记得加-h 确保SAM文件中含有表头
samtools view -h aln.bam > aln2.sam
```

### 索引一下BAM文件

很多分析，或者可视化BAM文件都需要你对其进行索引
```
samtools index aln.bam
```

### 过滤/提取BAM文件中比对不上的reads

这里需要你对SAM文件中的flags有一定了解，这方面很多文章也有介绍了，就不深入讲了，简单的把常见的贴给大家看：
```
samtools flags

About: Convert between textual and numeric flag representation
Usage: samtools flags INT|STR[,...]

Flags:
        0x1     PAIRED        .. paired-end (or multiple-segment) sequencing technology
        0x2     PROPER_PAIR   .. each segment properly aligned according to the aligner
        0x4     UNMAP         .. segment unmapped
        0x8     MUNMAP        .. next segment in the template unmapped
        0x10    REVERSE       .. SEQ is reverse complemented
        0x20    MREVERSE      .. SEQ of the next segment in the template is reversed
        0x40    READ1         .. the first segment in the template
        0x80    READ2         .. the last segment in the template
        0x100   SECONDARY     .. secondary alignment
        0x200   QCFAIL        .. not passing quality controls
        0x400   DUP           .. PCR or optical duplicate
        0x800   SUPPLEMENTARY .. supplementary alignment

samtools flags 16
0x10    16      REVERSE
```
提取或者过滤：
```
### 过滤
samtools view -h -F 4 -b aln.bam > aln_only_mapped.bam

### 提取
samtools view -h -f 4 -b aln.bam > aln_only_unmapped.bam
```

### 只提取BAM文件中比对到特定区域的信息

这里可以巧妙的利用`samtools view`，将生成的文件导出到新得BAM文件中:

```
samtools view -b aln.bam 1000000:1000-1300 > aln_1000_1300.bam
```
另外你还可以提供一个bed文件，根据bed文件的坐标，提取你所感兴趣的reads:

```
cat my.bed 
1000000 1000    1300
1000000 2000    2300

samtools view -L my.bed aln.bam
6144:733        147     1000000 1033    60      100M    =       733     -400    CCCTCCGGGGAGGGGATCTATTAGGAAAGATTCGAGTCGGTACGTTGTATGGAACGGTTAGCACCGCCATATATCAGATTCAAAATTATCTAGGGTTTCA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
7897:1206       99      1000000 1206    60      100M    =       1506    400     TTGCCGTGTACTGGTTGAGCCATACGCAAAATTGGGATCACATTTGTATTGACGCAGGAAAACTGATGCCCGTATCTCCAGAGGTACCAGAGCGACCTGG    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
9588:1278       99      1000000 1278    60      100M    =       1578    400     TATCTCCAGAGGTACCAGAGCGACCTGGACTTCAGCCAGGGAGATAGAGTATCACACTGAAGGACGGTAGCCGATAGATGAGCTGGACCTTGGATCTACT    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
10:1630 147     1000000 1930    60      100M    =       1630    -400    CCACCCACTACAGTAAATGAGAGAAAACGGATTAAGGTTCTAACCCTTAGCGATGACCGTTCTCACGGGTGTCTAGATACCGAGTAGAACCAACCAGCAC    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
9137:1657       147     1000000 1957    60      100M    =       1657    -400    CGGATTAAGGTTCTAACCCTTAGCGATGACCGTTCTCACGGGTGTCTAGATACCGAGTAGAACCAACCAGCACCCAGCATTAAACCACAGAAGCACCGTT    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
5893:1667       147     1000000 1967    60      100M    =       1667    -400    TTCTAACCCTTAGCGATGACCGTTCTCACGGGTGTCTAGATACCGAGTAGAACCAACCAGCACCCAGCATTAAACCACAGAAGCACCGTTGGTTTCTTTA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
3617:1997       99      1000000 1997    60      100M    =       2297    400     GGTGTCTAGATACCGAGTAGAACCAACCAGCACCCAGCATTAAACCACAGAAGCACCGTTGGTTTCTTTAATAGCTTCAACGCCGGCCATATCAATAAAA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
5736:1773       147     1000000 2073    60      100M    =       1773    -400    TCAACGCCGGCCATATCAATAAAAGGCTGTCGCCACATCCGTGGGTGCAAACTGAGCAGCCATGAACCTGGAAACGTTGTAGCCATGAAGATAATGCATA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
3123:2230       99      1000000 2230    60      100M    =       2530    400     CGGCACGTGCTGTCAGTATATAGTGTCGCTCATCAGGGAGGTACACCACGCGGTGGAGCATCCACGCTTTTCCCCATCTTCTATTACCTCGGCGGGGAAA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
4758:2263       99      1000000 2263    60      100M    =       2563    400     CAGGGAGGTACACCACGCGGTGGAGCATCCACGCTTTTCCCCATCTTCTATTACCTCGGCGGGGAAACAGGTAGATATGGGGGTTGGCTTGTGCAAGATA    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
3617:1997       147     1000000 2297    60      100M    =       1997    -400    TTTTCCCCATCTTCTATTACCTCGGCGGGGAAACAGGTAGATATGGGGGTTGGCTTGTGCAAGATACAATTCGATAGTTGCGGGGGCTTAGATCGGCGTG    JJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJJ  NM:i:0  MD:Z:100        AS:i:100        XS:i:0
```

### 随机提取BAM文件

有时候你可能想随机提取BAM文件进行一些测试:

```
### -s 0.5意味着提取一半的reads
samtools view -s 0.5 -b aln.bam > aln_random.bam
```

### 将BAM文件转化成fastq文件

可以使用两种方法:

```
### bedtools
bedtools bamtofastq -i aln.bam -fq aln.fq

### samtools
samtools fastq -1 aln_1.fq -2 aln_2.fq aln.bam
```

### 比对信息的统计

使用`samtools flagstat`可以获得简单的比对信息:

```
samtools flagstat aln.bam

20000 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
20000 + 0 mapped (100.00% : N/A)
20000 + 0 paired in sequencing
10000 + 0 read1
10000 + 0 read2
20000 + 0 properly paired (100.00% : N/A)
20000 + 0 with itself and mate mapped
0 + 0 singletons (0.00% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

如何你想获取更详细的信息可以使用`samtools stats`:

```
samtools stats aln.bam | grep ^SN

SN      raw total sequences:    20000
SN      filtered sequences:     0
SN      sequences:      20000
SN      is sorted:      1
SN      1st fragments:  10000
SN      last fragments: 10000
SN      reads mapped:   20000
SN      reads mapped and paired:        20000   # paired-end technology bit set + both mates mapped
SN      reads unmapped: 0
SN      reads properly paired:  20000   # proper-pair bit set
SN      reads paired:   20000   # paired-end technology bit set
SN      reads duplicated:       0       # PCR or optical duplicate bit set
SN      reads MQ0:      0       # mapped and MQ=0
SN      reads QC failed:        0
SN      non-primary alignments: 0
SN      total length:   2000000 # ignores clipping
SN      bases mapped:   2000000 # ignores clipping
SN      bases mapped (cigar):   2000000 # more accurate
SN      bases trimmed:  0
SN      bases duplicated:       0
SN      mismatches:     0       # from NM fields
SN      error rate:     0.000000e+00    # mismatches / bases mapped (cigar)
SN      average length: 100
SN      maximum length: 100
SN      average quality:        41.0
SN      insert size average:    400.0
SN      insert size standard deviation: 0.0
SN      inward oriented pairs:  10000
SN      outward oriented pairs: 0
SN      pairs with other orientation:   0
SN      pairs on different chromosomes: 0
```


### 比较两个BAM文件

有时候你想详细看看两个个体比对到用一个参考基因组的具体情况,这里要借助工具bamCompare:

```
###这些bam文件，在比较之前，都需要进行index和sort
bamCompare -b1 aln.bam -b2 aln2.bam -of bigwig -o aln_compare.bw
```

介绍大概到这里，还有很多很多功能没有在这里提到，具体的大家可以回到其对应的工作手册进行查阅。

参考资料：

 1. https://github.com/davetang/learning_bam_file

  [1]: http://static.zybuluo.com/lakesea/9u2lexp7yrdih3s0xzca4vw5/image.png
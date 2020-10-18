# Dsuite: 从VCF文件中快速计算D-统计量和亲缘相关性的工具

标签（空格分隔）： 生信工具

---

### 工具背景介绍

D统计量（也称为ABBA-BABA统计量）和相关统计量通常用于评估种群或紧密相关物种之间的是否存在基因流。当前计算D统计量的工具大多数都需要该工具自定义特定的文件格式，并且不便于评估包含许多种群或物种的基因流。

**Dsuite**是一种快速的通过C++实现的工具。可以对数十个甚至数百个种群或物种的所有组合进行D统计量进行计算，并且可以直接使用常见的变异（VCF）文件作为输入。此外，该程序可以估算混合物分数，并提供基因渗入是否仅限于特定基因座的证据。因此，**Dsuite**非常适用于评估跨大型基因组数据集的基因流。


### 工具下载与安装

在编译安装之前，首先你要有GCC并且其版本需要大于4.9.0。

该工具可以通过github进行直接下载:

```
git clone https://github.com/millanek/Dsuite.git
cd Dsuite
make
```
编译好后可以通过`./Build/Dsuite`来测试命令能否成功执行。

与其他类似的工具相比，Dsuite在运行时间和消耗内存方面都有更好的表现。
![][1]

### 输入文件

主要的输入文件有两个:

 1. VCF文件，可以是被压缩的格式(gzip或者bgzip)。文件可以含有multiallelic 位点 and indels，但是只有biallelic 位点才会用于分析。
 2. 群体/物种图(SETS.txt)，一个文本文件，每行代表一个个体，和其所属的物种/种群名称，如下所示。

```
Ind1    Species1
Ind2    Species1
Ind3    Species2
Ind4    Species2
Ind5    Species3
Ind6    Outgroup
Ind7    Outgroup
Ind8    xxx
...     ...
IndN    Species_n
```

这里最少要设置一个`Outgroup`外群物种。如果你想完全忽略某个个体，可以使用`xxx`将其屏蔽（不需要子集hua VCF文件了）。


### 可选的输入文件

 1. 进化树文件(newick)格式，这棵树需要有与物种/种群名称相对应的叶子标签。分支长度可以有，但是分析中不使用。具体的例子如下：

```
(Species2,(Species1,(Species3,Species4)));
(Species2:6.0,(Species1:5.0,(Species3:3.0,Species4:4.0)));
```

 2. 三重组合文件,每一行三个个体/物种，由标签按顺序分隔P1 P2 P3

```
Species1    Species2    Species3
Species1    Species4    Species2
...         ...         ...
```
 
 测试文件下载:
 
```
###VCF file
wget http://cichlid.gurdon.cam.ac.uk/Malinsky_et_al_2018_LakeMalawiCichlids_scaffold_0.vcf.gz
 
###群体文件SETS.txt)

wget http://cichlid.gurdon.cam.ac.uk/sets.txt
```

### 工具使用


#### Dsuite Dtrios -为所有可能的种群/物种三重组合计算D统计量（ABBA-BABA）

使用测试数据进行测试:

```
~/biosoft/Dsuite/Build/Dsuite Dtrios  Malinsky_et_al_2018_LakeMalawiCichlids_scaffold_0.vcf.gz sets.txt
```

生成下列的文件：

```
-rw-r-----  1 hhu hhu  49M Oct  8 11:51 sets__combine_stderr.txt
-rw-r-----  1 hhu hhu 5.6M Oct  8 11:51 sets__combine.txt
-rw-r-----  1 hhu hhu 5.5M Oct  8 11:51 sets__Dmin.txt
-rw-r-----  1 hhu hhu 5.5M Oct  8 11:51 sets__BBAA.txt
```
简单看看包含D统计量的sets_Dmin.txt文件,可以看到每个三重组合所对应的D值与其P-value:

```
head sets__BBAA.txt

==> sets__BBAA.txt <==
P1      P2      P3      Dstatistic      p-value
Alticorpus_macrocleithrum       Alticorpus_geoffreyi    A_calliptera    0.00562169      0.388387
Aulonocara_minutus      Alticorpus_geoffreyi    A_calliptera    0.0084396       0.247183
Alticorpus_geoffreyi    Aulonocara_steveni      A_calliptera    0.0283765       0.0798469
Alticorpus_geoffreyi    Aulonocara_stuartgranti A_calliptera    0.0240418       0.0681949
Aulonocara_yellow       Alticorpus_geoffreyi    A_calliptera    0.0201788       0.0998002
Alticorpus_geoffreyi    Buccochromis_nototaenia A_calliptera    0.0455587       0.00370416
Alticorpus_geoffreyi    Buccochromis_rhoadesii  A_calliptera    0.0741405       5.45716e-05
Alticorpus_geoffreyi    Champsochromis_caeruelus        A_calliptera    0.0666998       0.000146784
Alticorpus_geoffreyi    Chilotilapia_rhoadesii  A_calliptera    0.0800385       2.43101e-05
```

#### Dinvestigate-对D统计量显着提高的三重奏进行后续分析：计算f4统计信息，以及沿基因组窗口中的f_d和f_dM

```
Dsuite Dinvestigate INPUT_FILE.vcf.gz SETS.txt test_trios.txt
```

介绍到这里，工具固然是好用，但是怎样解析生成的结果，怎样从结果中提取有用的信息是更重要的下一步，后面的推文会通过阅读一些文章和大家分享一些如何解读基因渗入相关的内容。


参考网站：

 1. Dsuite的GitHub：https://github.com/millanek/Dsuite
 2. Dsuite的初稿: https://www.biorxiv.org/content/10.1101/634477v1

 
 
 


  [1]: http://static.zybuluo.com/lakesea/2p30p1x66ztjlchyqeylztx0/%E6%8D%95%E8%8E%B7.PNG
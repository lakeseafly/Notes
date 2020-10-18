# 生信小白学习系列：使用MaSuRCA对拟南芥进行组装的实战训练(2)

标签（空格分隔）： 生信工具

---

继上一次内容讲完组装的基本知识后，大家对其理论知识都有了一定程度的了解。当然，如果你还没有阅读或者忘记了上一讲的内容，请不要犹豫先点开一下连接，先进行回顾一番：

这周开始会给大家一起进行实战练习，使用MaSuRCA进行拟南芥基因组组装，如果你有合适的服务器，欢迎跟着我操作一波起来。


###MaSuRCA组装软件介绍


MaSuRCA是由马里兰大学研究人员写的全基因组装软件。它结合了de Bruijn图和Overlap-Layout-Consensus（OLC）方法的效率。MaSuRCA可以组装仅包含来自Illumina测序的短reads或短reads和长reads的混合物的数据集合（Sanger，454，Pacbio和Nanopore）。听起来感觉棒棒哒。到底好不好用，先来扒一扒原理。

首先，用错误率低的illumina短reads来搭建较长的super-reads,组成一个15-mer的备用数据库；然后，以错误率高的PacBio长reads作为模板，使用备用数据库中的super-reads进行比对，super-reads连接并且延长，组成更长的pre-mega-reads,不连续的super-reads将会被丢弃;  最后，从pre-mega-reads中挑选出最终需要的mega-reads，用来组装基因组。听起来是不是比拼206块人骨容易多了？没听明白还在懵逼的科科们，快醒醒奥运会都结束了，赶紧去瞅瞅下图。


![][1]








###MaSuRCA软件安装

首先去官网下载安装包，请根据系统选择版本MaSuRCA-X.X.X-Y.tar.gz，这里推荐是要下载3.3.1之后的版本，因为3.3.1之后版本修改了之前所有版本的一个bug（执行gapfilling的do_consensus.sh时会报错，需要手动进行修复），最新版本是前几天出的3.3.2，进一步提升了功能，但是我的实战是基于比较稳定的3.3.1版，所以3.3.1以上的都可使用进行下面的实战。

下载链接如下：https://github.com/alekseyzimin/masurca/releases
对系统的安装要求可以在其github中找到：https://github.com/alekseyzimin/masurca

对硬件的要求和估计运行时间：

•	细菌(up to 10Mb): 16Gb RAM, 8+ cores, 10Gb disk space，<1小时
•	昆虫 (up to 500Mb): 128Gb RAM, 16+ cores, 1Tb disk space，1-2天
•	禽类，小型植物 (up to 1Gb): 256Gb RAM, 32+ cores, 2Tb disk space，4-5天
•	哺乳动物 (up to 3Gb): 512Gb RAM, 32+ cores, 5Tb disk space，15-20天
•	大型植物基因组 (up to 30Gb): 1Tb RAM, 64+cores, 10Tb+ disk space，60-90天


安装过程，只要系统的requirement都是满足的，安装简单一步到位
```
tar xvzf MaSuRCA-3.3.1.tar.gz
cd MaSuRCA-3.3.1
bash install.sh
```
  [1]: http://www.biotrainee.com/data/attachment/forum/201609/23/112435vxs933igo7m3u33k.jpg
  
  
###测试数据的下载


拟南芥可能是一个model species， 其基因组大小约为135Mb，非常适合作为测试数据，这里我们使用拟南芥的公众数据进行组装。

**从SRA中下载测试数据**

NCBI中有很多的测试数据，这里我们这挑选其中一小部分进行该实战演习。其中下载数据需要用到sra-tools，大家可以自行到官网进行下载(https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/)，这里就不岔开说了(如何大家有很多反馈不懂如何通过SRA下数据，以后可以出一个专门是如何通过sra-tools下载数据的专题)。


```
###创建raw data的文件夹
mkdir -p masurcatutorial/raw-data
cd masurcatutorial/raw-data

###创建一个需要下载文件的list

vi srr.ids

###将下面的文件名复制粘贴到srr.ids中
SRR3156163
SRR3157034
SRR3156160

###通过sra-tools里面的fastq-dump下载所需文件

##下载文件这里可能需要个小技巧，因为fastq-dump会默认的下载到你的HOME directory那里，但是很多时候，特别是使用超算，或者公共服务器的用户，HOME directory都是有限制空间大小的，一般需要在work directory那里下载数据，这里有个小小的把戏，偷换一下HOME的变量，使用存储量更大的地方进行下载

export HOME=xxx/xxx/path/to/more/space/

prefetch --option-file srr.ids -X 100000000

###将sra格式的数据转换成fastq格式
fastq-dump --split-3 --outdir /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra --defline-qual '+' SRR3156160.sra
fastq-dump --split-3 --outdir /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra --defline-qual '+' SRR3156163.sra
fastq-dump --split-3 --outdir /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra --defline-qual '+' SRR3157034.sra
```
这里可以开始组装了，masurca的组装需要一个配置文件，里面需要填入你的选用的library的名称，还有该library对应的插入片段的长度。
```
vi masurca.conf

###将以下信息填入masurca.conf中
DATA
PE = pb 250 50 /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra/SRR3157034_1.fastq  /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra/SRR3157034_2.fastq
JUMP = ja 8000 1600 /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra/SRR3156163_1.fastq  /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra/SRR3156163_2.fastq
PACBIO = /scratch/pawsey0149/hhu/test/masurcatutorial/raw-data/ncbi/public/sra/SRR3156160.fastq
END

PARAMETERS
GRAPH_KMER_SIZE = auto
USE_LINKING_MATES = 0
LIMIT_JUMP_COVERAGE = 300
CA_PARAMETERS =  cgwErrorRate=0.15
KMER_COUNT_THRESHOLD = 1
NUM_THREADS = 16
JF_SIZE = 200000000
SOAP_ASSEMBLY=0
END
```

填好后就可以准备开始运行了：

```
masurca masurca.conf 

###敲完上面的命令，系统就给你检测一系列对应的依赖包，还有其相应工作的环境
Verifying PATHS...
jellyfish OK
runCA OK
createSuperReadsForDirectory.perl OK
nucmer OK
mega_reads_assemble_cluster.sh OK
creating script file for the actions...done.
execute assemble.sh to run assembly

###最后会生成一个assemble.sh， 然后简单的敲一下下面的命令就可以开始组装了。

bash assemble.sh 

```



组装时候会生成很多文件，可能这里我选用的文件还是有点太大了，运行了一天还没有完成，这里就把参考资料里已经完成的进度表拿出来展示,最后的组装好的文件会存储于CA.xxxxx/final.genome.scf.fasta里，进度表最后也会有该组装的一些参数(N50,平均长度等，关于组装的评测会再将来的文章再讲)，让你快速知道组装的效果好不好：

```
 [Fri Oct 26 08:29:01 EDT 2018] Processing pe library reads
  [Fri Oct 26 08:30:02 EDT 2018] Average PE read length 249
  [Fri Oct 26 08:30:02 EDT 2018] Using kmer size of 127 for the graph
  [Fri Oct 26 08:30:02 EDT 2018] MIN_Q_CHAR: 33
  [Fri Oct 26 08:30:02 EDT 2018] Creating mer database for Quorum
  [Fri Oct 26 08:31:11 EDT 2018] Error correct PE
  [Fri Oct 26 08:36:18 EDT 2018] Estimating genome size
  [Fri Oct 26 08:37:06 EDT 2018] Estimated genome size: 147278815
  [Fri Oct 26 08:37:06 EDT 2018] Creating k-unitigs with k=127
  [Fri Oct 26 08:41:11 EDT 2018] Computing super reads from PE
  [Fri Oct 26 08:46:02 EDT 2018] Using CABOG from /opt/spack/opt/spack/linux-centos7-x86_64/gcc-4.8.5/masurca-3.2.8-hgdshdatclkrv7kct6kqgfb3fa536iji/bin/../CA8/Linux-amd64/bin
  [Fri Oct 26 08:46:02 EDT 2018] Running mega-reads correction/assembly
  [Fri Oct 26 08:46:02 EDT 2018] Using mer size 15 for mapping, B=17, d=0.029
  [Fri Oct 26 08:46:02 EDT 2018] Estimated Genome Size 147278815
  [Fri Oct 26 08:46:02 EDT 2018] Estimated Ploidy 1
  [Fri Oct 26 08:46:02 EDT 2018] Using 28 threads
  [Fri Oct 26 08:46:02 EDT 2018] Output prefix mr.41.15.17.0.029
  [Fri Oct 26 08:46:02 EDT 2018] Pacbio coverage <25x, using the longest subreads
  [Fri Oct 26 08:46:02 EDT 2018] Reducing super-read k-mer size
  [Fri Oct 26 08:48:51 EDT 2018] Mega-reads pass 1
  [Fri Oct 26 08:48:51 EDT 2018] Running locally in 1 batch
  [Fri Oct 26 08:49:38 EDT 2018] Mega-reads pass 2
  [Fri Oct 26 08:49:38 EDT 2018] Running locally in 1 batch
  [Fri Oct 26 08:49:42 EDT 2018] Refining alignments
  [Fri Oct 26 08:49:43 EDT 2018] Joining
  [Fri Oct 26 08:49:43 EDT 2018] Gap consensus
  [Fri Oct 26 08:49:43 EDT 2018] Warning! Some or all gap consensus jobs failed, see files in mr.41.15.17.0.029.join_consensus.tmp, proceeding anyway, to rerun gap consensus erase mr.41.15.17.0.029.1.fa and re-run asse
  mble.sh
  [Fri Oct 26 08:49:43 EDT 2018] Generating assembly input files
  [Fri Oct 26 08:49:44 EDT 2018] Coverage of the mega-reads less than 5 -- using the super reads as well
  [Fri Oct 26 08:49:49 EDT 2018] Coverage threshold for splitting unitigs is 15 minimum ovl 250
  [Fri Oct 26 08:49:49 EDT 2018] Running assembly
  [Fri Oct 26 08:58:46 EDT 2018] Recomputing A-stat
  recomputing A-stat for super-reads
  [Fri Oct 26 09:01:15 EDT 2018] Mega-reads initial assembly complete
  [Fri Oct 26 09:01:15 EDT 2018] No gap closing possible
  [Fri Oct 26 09:01:15 EDT 2018] Removing redundant scaffolds
  [Fri Oct 26 09:02:07 EDT 2018] Assembly complete, final scaffold sequences are in CA.mr.41.15.17.0.029/final.genome.scf.fasta
  [Fri Oct 26 09:02:07 EDT 2018] All done
  [Fri Oct 26 09:02:07 EDT 2018] Final stats for CA.mr.41.15.17.0.029/final.genome.scf.fasta
  N50 32439
  Sequence 100640646
  Average 16714.9
  E-size 43482.6
  Count 6021
```

使用MaSuRCA进行组装就到这里差不多了，有兴趣或者想学的粉丝们，可以动起来手来试一试该工具。除了该教程提到的东西外，MaSuRCA还有很多其它的参数还没有提及到，具体内容大家可以参考它的使用手册，以liaoj



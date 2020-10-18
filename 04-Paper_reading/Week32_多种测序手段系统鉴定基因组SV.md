# Week32:多种测序手段系统鉴定基因组SV

标签（空格分隔）： 文献解读

---

**题目:** Integrative detection and analysis of structural variation in cancer genomes


**摘要:**
结构变异（SV）可以通过多种机制促进肿瘤发生。尽管它们的非常重要，癌症基因组中的SV的识别仍然具有挑战性。这里，作者开发了一个结合光学图谱、Hi-C和全基因组测序的三种数据的流程，可以系统地检测正常或癌样本和细胞系的各种SV。这种识别每种方法的独特优势，并证明只有综合三种测序手段才能全面地识别基因组中的SV。通过结合HI-C和光学图谱，可以阐明复杂的SV。此外，作者观察到广泛的结构变异事件，影响非编码序列的功能，包括远端调控序列的缺失，DNA复制时间的改变，以及新的三维染色质结构域的建立。作者分析这些结果表明非编码区域的SV可能是癌症基因组中未被重视的突变驱动因子。

**Results:**

**三种技术的基本对比**

![][1]

研究团队首先在癌症和健康细胞系中，对Hi-C技术、WGS和光学图谱技术进行了分析评估。研究发现这3种方法都可以在几十种癌细胞系中，以不同的规模和分辨率获取新的或已公布的SV信息，但这些方法都只能检测到部分SV。研究人员强调，Hi-C方法在组装基因组、单倍体分型方面有优势，但在检测小于1Mb DNA片段SV方面能力有限；WGS在检测SV方面具有较高的分辨率，但人类基因组存在49%的重复序列，WGS的短序列往往无法正确映射到重复序列，限制了SV检测。此外，WGS会将DNA打断成小于500bp的片段，破坏了SV之间的连续性，无法检测复杂SV；光学图谱技术在检测复杂SV和解析局部基因组结构方面表现优异，但无法检测到微小片段的缺失和插入。

**三种技术的综合结合**
![][2]

为解决以上问题，研究团队决定将Hi-C技术、WGS和光学图谱技术结合，取长补短、优势互补。在30多种癌细胞系的复杂SV检测中，研究人员借助光学图谱技术构建局部基因组，利用Hi-C技术揭示的SV连续性关系，将多个局部基因组组成单倍体，WGS则进行了SV位点的精确检测定位，最终在单倍型水平完成局部基因组组装，还原了复杂SV的全貌。此外，研究团队还首次提出了基于Hi-C数据的算法，该算法可在一般的Hi-C测序深度进行，用于全基因组SV检测。经过在30多个癌症细胞系中进行SV检测实验，研究人员验证了该算法的可靠性。

借助新综合检测方法，研究人员发现了许多由SV导致的三维基因组改变的实例，例如拓扑结构域的形成或解散，表明在肿瘤发生中，SV导致的基因失调起到关键作用。他们还观察到了影响非编码序列功能的广泛SV事件，包括远端调控序列的缺失，DNA复制时间的改变等。研究表明，非编码SV可能是癌症基因组中未被充分认识的另一驱动因子。

**Methods**

**总的思路**

 利用七个癌细胞系中产生WGS数据，平均覆盖率＞30×，构造了一个内部流程结合了LUMPY, Delly, and Control-FREEC三个软件，用于初始SV检测，然后执行数据过滤；光学图谱，平均覆盖率约为100×。使用RefAligner 6119 (Bionano Genomics) and pipeline 6498进行从头组装和SV检测，并进行进一步数据过滤；最后作者开发了一种使用Hi-C识别重排事件的，包括易位，倒位、缺失和串联复制的算法；经过比较和合并每个测序平台的结果，确定了数千个插入和缺失（＞50 bp），数百个串联重复和染色体间易位，数十个倒位。
 
 **关于WGS的SV calling**

目前有比较多WGS的数据，重点关注一下该文章所使用的方法。


**Delly pipeline** 

1.  BWA-MEM 用来比对paired-end 的reads到ref上
2.  过滤，只保留Mapping quality 20 以上的比对上的reads
3. Delly用来call SV,包括：deletions, inversions, tandem duplications, insertions, and interchromosomal translocations.

**SpeedSeq pipline**

1.  BWA-MEM 用来比对paired-end 的reads到ref上
2.  使用samblaster去掉重复比对的片段
3.  取出不连续分裂的片段去做SV calling
4.  使用LUMPY (speedseq sv -g -t 64 -x)进行SV calling

**Copy number profiles** 
使用Control-FREEC

**SV filtering**

1. 至少有三个split reads深度在这个SV的位置上
2. LUMPY和Delly的结果merge在一起，两个工具重合calling的SV保留
**Deletion:**
如果该两个工具的结果互相之间有50%的重合。

Additional filtration was applied to specific types of SVs. For deletions, we
removed deletions that had at least 50% reciprocal overlap (RO ≥ 50%) with known gap regions (± 50 bp), or at least a 1 bp overlap with centromere regions (± 1 kb). Recurrent deletions that were larger than 1 Mb and presented in more than one cell line with an RO ≥ 99.9% were removed. Large deletions (≥ 100 kb) that did not show consistent decrease of read depth compared with adjacent regions were also removed (less than one difference of read depth between deletions and flanking 10-kb regions). For
**Inversions**
如果该两个工具的结果互相之间有90%的重合。

recurrent inversions that were longer than 100 kb and were present in more than one cell line (defined by RO ≥ 99.9%) were removed.
**Translocations**
paired break ends mapped within ± 50 bp of each other and if the strand of the break ends matched。

recurrent translocations that were present in more than one cell line (defined by both break ends being within ± 50 bp) were filtered out.

minimal number of supporting split reads paired-end
reads (SR + PE)，Due to high sequencing coverage (~80× ) in the LNCaP sample, we only kept translocations with at least 15 SRs (PE + SR)，(coverage of 50× ), since they are diploid, we used a more stringent filter of 20 SRs

**insertions**
使用delly来发掘，lumpy不支持该SV的发现



###小结

随着测序技术的发展，SV会是以后研究的一个重点，无论是人类还是植物或者其他。该文章提供了一个很好的例子如何通过不同的测序技术去定位SV，然后还有一系列相关的功能研究。文章所提到的工具，数据都是很好的学习样本。




  [1]: https://mmbiz.qpic.cn/mmbiz_png/N3X4LBoaQjUicJC0nVyLDuQrKMgJE9GcWGJDZdmPjDRwrDSa4fRvEqNvYJicP589ut7GYa55dLCqOmt9I3Tt8x9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/DMKW2dzPflKEFaMrqldDia7JLCwW3CskyqzWJUmDTa3lFiaCvpxVoQ20ath0WWvYZuuy9btzHCtrdAvfkYBZdF3g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
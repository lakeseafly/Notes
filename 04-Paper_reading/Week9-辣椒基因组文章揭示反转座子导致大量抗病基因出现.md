# Week9-辣椒基因组文章揭示反转座子导致大量抗病基因出现

> 简单给大家解读一篇2017年在Genome Biology 发表的一篇文章，顺带介绍一些相关的基本知识。


##题目：New reference genome sequences of hot pepper reveal the massive evolution of plant disease-resistance genes by retroduplication

[原文链接][1]

##文章研究重点

 1. 辣椒（Capsicum）物种形成和进化
 2. LTR主导的反转座子辅助导致产生了很多NLRs
 3. 茄科(Solanaceae）中抗病基因的出现的进化机制
 4. 朝天椒（Baccatum）中跟炭疽病相关的基因的进化历程


##Background

###Summary
 
转座因子是主要的进化力量，可导致新的基因组结构和物种多样化。转座因子在扩大核苷酸结合和富含亮氨酸重复序列蛋白（NLR）这一主要抗病基因家族中的作用在植物中尚未被探索。
研究者报道了两种高质量的de novo基因组（辣椒辣椒和C. chinense）和一个改良的辣椒参考基因组（C. annuum）。动态基因组重排涉及染色体3,5和9之间的易位在风铃辣椒和其他两种辣椒之间的比较中检测到。 athila LTR反转录转座子，gypsy superfamily 的扩增导致朝天椒（C.Baccatum）的基因组扩增。通过对基因和重复序列进行深入的全基因组比较，发现LTR-反转录转座子介导的重新引入大大增加了NLR的拷贝数。此外，重新引入的NLR在被子植物中普遍存在，并且在大多数情况下是有特定谱系的特征。该研究表明retroduplication 在NLR基因大量出现中起到了重要的作用。


###基础知识介绍

**Transposable elements（TEs）:转座子**

![此处输入图片的描述][2]
转座子是一类DNA序列，是真核生物基因组中普遍存在的可移动和自主复制的DNA重复序列。它们能够在基因组中通过转录或逆转录，在内切酶的作用下，在其他基因座上出现。转座子的这种行为，与假基因（Pseudogene）的出现颇有相似甚至相同之处。有些科学家将后者视为“基因化石”，是透视物种进化的痕迹之一。转座子的发现，证明了基因组并不是一个静态的集合，而是一个不断在改变自身构成的动态有机体。

**Nucleotide-binding domain leucine-rich repeat（NLR):
核苷酸结合结构域富含亮氨酸的重复序列**

核苷酸结合结构域富含亮氨酸重复序列（NLR）蛋白质是最近发现的细胞内病原体和危险信号传感器家族。 NLR已成为动物先天免疫的重要贡献者。这些基因的生理影响越来越明显，变异家族成员与一系列炎症性疾病的遗传关联强调了这些基因的生理影响。 NLR基因突变与自身炎症疾病的关联表明这些基因在体内炎症中的重要功能。

**Long terminal repeat retroposons (LTR-Rs):LTR反转录转座子**

LTR反转录转座子是I类转座元件，其特征在于存在长直接重复序列（LTR），其直接位于内部编码区域的侧翼。作为逆转录转座子，它们通过逆转录它们的mRNA并将新产生的cDNA整合到另一个位置而动员起来。它们的逆转录转座机制与逆转录病毒共享，不同之处在于大多数LTR-逆转录转座子不形成离开细胞的感染性颗粒，因此仅在其起源的基因组内复制。

**Retrogenes： 逆转座复制的基因**

从逆转录复制的DNA基因转录的RNA。

 - 与亲本基因相比，丢失了内含子
 - 基因组中有3' PolyA存在
 - 侧翼有直接重复（Direct Repeat）


##Results

###从头测序，组装和注释（Capsicum）

研究者针对Caosicum baccatum 和C.chinese 两个物种进行了测序，利用的平台是hiseq 2500，构建的是200bp-10Kb的文库。

对于物种基因组大小进行评估，Caosicum baccatum为3.9Gb，C.chinese为3.2Gb。组装的结果分别为3.2Gb和3.0Gb，完整度大约为83%和94%。组装的水平为scaffold N50 2Mb，contig N50 为39Kb，另一个scaffold N50为3.3Mb，contig N50 为50Kb。基因预测大概在35000个基因左右。另外通过GBS构建F2待群体构建了一套高密度的遗传图，将scaffold划分到了12条染色体，其中挂在率分别为87%和89%。另外还优化了一下Annuum基因组。基因组的重复序列比例占到了85%，而仅Ty3-gypsy超家族就占到了基因组的一半，在gypsy超家族中，del元件占据了最大的比例，athila亚家族在三个辣椒的种中表现出了品种特异性。

###Capsicum物种形成和进化

通过辣椒跟其他物种构建进化树发现，三个辣椒之前分化也存在差异。首先分化在Baccatum和其他物种之上。
研究者进行了基因组间的结构比较，并检测了辣椒基因组中的易位现象。结果显示染色体3,5和9显示出使Baccatum与其他两个物种区分的易位。辣椒属物种与两种茄属物种的共线性比较揭示，9号染色体长臂上的远端区域在Baccatum中保守，但易位于Annuum和Chinense的共同祖先中的染色体3的短臂。

![此处输入图片的描述][3]
 
 
为了比较辣椒基因组中的LTR-R插入模式，研究者对每个装配的基因组中鉴定了全长LTR-R并估计它们的插入时间。Baccatum的LTR-R活性峰在其形成时间1-2 MYA附近出现。特别是，在估计的物种形成时间周围，athila家族在Baccatum中高度扩增，表明该物种在物种形成后可能在Baccatum中爆发性增加。在Chinense，我们观察到LTR-Rs最近的增殖。基因重复是通过创造新基因在物种间产生功能多样性的主要机制。研究者检测到最近的基因复制事件，并在物种形成过程中和形成后，对三个辣椒基因组中的重复基因的特征进行了表征对比。总的来说，Baccatum谱系中的重复事件在物种形成期间和之后特别频繁。特别是，NLRs在Baccatum中的最后0-2 MYA中广泛扩增，并且最近在另外两个辣椒中扩增。总之，这些结果表明染色体重排，特定LTR-R的积累和差异基因重复已经促成了辣椒基因组的多样化。
![此处输入图片的描述][4]

###通过LTR主导的反转座子辅助导致产生了很多NLRs

研究者对NLR的侧翼序列进行了分析，发现有13%都是LTR逆转座带来的基因重复，而且其结构还是完整的。对于13%的NLR，70%是CNL-G2类型的（辣椒里扩张的一个类型），72%是位于非自主LTR区域内。另外，逆转座R基因和正常R基因都受纯化选择。

###Solanaceae中抗病基因的出现的进化机制
L, I2, R3a这三个基因，分别是辣椒、番茄、土豆中的抗性基因，也是逆转座NLR基因，研究者发现他们也具有有高度共线性关系（在chr11上），所以推测可能是早期发生了逆转座重复然后又在各个枝系（lineage）进行了分化。研究者还研究了L的进化历史，根据相似性鉴定了P1-P6六个假定的亲本基因，根据L和P1比较，推测分歧时间可能在8.9mya。因为L只有一个内含子、侧翼有直接重复、3'端有polyA尾巴，所以研究者认为L是由LINE驱动的逆转座重复产生的。


##方法

 - 使用SOAP-denovo2 还有SSPACE，Platanus还有Gapcloser辅助，进行组装。
 - 基因组组装经典的EVM pipeline：
Transcript evidence：Tophat+Cufflinks （使用ISGAP 尽心准确提取coding sequences）
Protein evidence： exonerate （拟南芥，番茄，土豆，胡椒ref）
Predicted evidence: AYGYSTYS 
Functional annotation：Blast Uniprot + Interpro scan 

 - 重复序列鉴定用到了RepeatModeler、LTRharvest(-maxlenltr 2000 and –similar 80)+LTRdigest。LTR亚家族的鉴定是和完整的LTR-R进行BLASTN比对(similarity > 90%)。
 - 共线性使用MCScanX鉴定，基因组之前的结构
 - 重复基因的鉴定：首先用OrthoMCL聚类，然后PRANK进行all-vs-all比对，并对每个比对计算一次Ka/Ks （KaKsCalculator），Ks使用R里面的hcluster进行聚类，分子进化速率r引用了一篇文献7，为6.96 × 10−9，算下来T=Ks/2r。
 - 物种间的分化时间计算方法：OrthoMCL聚类->PRANK比对单拷贝基因(-f = nexus -codon)->BEAST估算分歧时间
 - LTR的DNA替换速率（K）使用paml里面的baseml估算的，公式还是T=Ks/2r，但这里的r用了一个更快的值：1.3 × 10−8，作者的理由是intergenic 区域进化比coding的区域有更高的替代进化率。

 
  [1]: https://genomebiology.biomedcentral.com/articles/10.1186/s13059-017-1341-9
  [2]: https://static.xmt.cn/3906931369169793329.png
  [3]: http://media.springernature.com/full/springer-static/image/art:10.1186/s13059-017-1341-9/MediaObjects/13059_2017_1341_Fig1_HTML.gif
  [4]: http://media.springernature.com/full/springer-static/image/art:10.1186/s13059-017-1341-9/MediaObjects/13059_2017_1341_Fig2_HTML.gif
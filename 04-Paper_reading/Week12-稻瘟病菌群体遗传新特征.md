# Week12-稻瘟病菌群体遗传新特征

> Rice blast稻瘟病菌是我接触生信的一个研究的对象，当初得到的数据还蛮不错，可惜由于当时还是生信生手，并没有好好利用。今天的这篇文章，从群体遗传的角度去研究rice blast，今天和大家简单解读学习一下这文章,研究一下如果你有一份好的数据，你怎样通过分析找到相应的。

**题目**

Population genomic analysis of the rice blast fungus reveals specific events associated with expansion of three main clades


**背景**

稻瘟病菌（Pyricularia oryzae syn. Magnaporthe oryzae）能侵染水稻引致稻瘟病，每年给世界水稻生产造成严重损失。但是稻瘟病菌群体复杂多样，毒性变异迅速，致使一个新的水稻抗瘟品种往往种植3-5年后就失去抗性，使得该病害防控十分棘手。因此，研究明确稻瘟病菌群体遗传多样性特点及其产生机制十分重要，也是制定稻瘟病生态防控策略的理论基础


**Methods**

常规方法大部分会，但是population genetics部分的还需要加以学习研究。

**genome sequencing, and annotation**

 - De novo sequence assembly: Workbench 7.0
 - evidence-based prediction: Exonerate 
 -  ab initio gene prediction: SoftBerry
 -  EVM 最终整合

**Single-nucleotide polymorphism discovery, identification of mating-related genes, and transposon element annotation**

 - SNP calling：Samtools + GATK （allele/depth <0.9, and filtered for multiple alleles with other parameters set to the default）
 -  genome-to-genome SNP calling  （MUMmer）


**Phylogenomic tree construction, population structure, and divergence time estimation**

 - phylogenomic tree:FastTree (with the approximately-maximum-likelihood model)
 - populational structure: Structure 2.3.4 with whole-genome SNPs, and K value selected by the Evanno method
 - estimate divergence time : BEAST (v1.8.3)
 - PCA : SNPs results by Taqssel 5.0
 - timescaled tree: FigTree

**Population genetics and genomics**

 - nucleotide diversity calculations: DNAsp
 - Ka/Ks values （在遗传学中， Ka / Ks比率 （或ω，dN / dS）是每个非同义位点的非同义替换数（Ka）与每个同义位点的同义替换数（Ks）之比，可以是用作作用于蛋白质编码基因的选择压力的指标。）：KaKs_Caculator 2.0
 - ecombination rate (ρ) calculation， nucleotide diversity, Tajima’s D, and Theta calculation：VariScan 
 - Fst values （ 群体间固定指数： 衡量群体中等位基因频率是否偏离遗传平衡论比例的指标，用来研究不同群体间的分化程度。其取值为0到1，0代表两个群体未分化，其成员间是完全随机交配的；1代表两个群体完全分化，形成物种隔离，且无共同的多样性存在。）：Vcftools
 - PAV： BLASTN 

**Results**
 该研究首次利用群体基因组学的方法对来自中国和其他国家、地区的100个稻瘟病菌田间菌株全基因组SNP位点进行分析，并构建系统发育进化树，结果发现这些菌株可以划分为3个亚群，每个亚群在世界各地均有不同程度的分布，暗示聚类结果与菌株来源地无关；但进一步研究发现这些菌株的聚类结果与其交配型密切相关，其中Clade 1包含Mat 1-1和Mat1-2两种交配型，Clade 2仅包含Mat1-2，Clade 3则仅包含Mat1-1。本研究还发现，水稻宿主不同（籼稻和粳稻）可能是导致Clade 2和Clade 3 群体交配型单一化的主要驱动力。研究结果将为不同地区稻瘟病菌群体特征研究并有针对性地进行水稻抗瘟育种和品种布局利用提供理论依据。
 
 
 **心得**
 
 当年我也有一份类似的数据100份的rice blast fungus， 当时主要做泛基因组的研究。可是并没有把重点放在call snp上。其实snp calling 确实最重要的一部分，像很多population的研究其实都是基于snp calling 的结果。日后，要开始学习population genomics 的相关知识，遗传驯化一直是genetics中很有趣的一些话题。本文找到了切入点，三个不同的clusters，然后将其联系到对应的mating type就写出了本文。
 
 做好snp calling，做基因驯化，遗传相关的一些分析，找到相关有趣的科学问题，事发好文章的关键。
 
 

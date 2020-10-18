# Week-29 辣椒的泛基因组

标签（空格分隔）： 文献解读

---

**题目：**Pan-genome of cultivated pepper (Capsicum) and its use in gene presence-absence variation analyses
[原文链接][1]


**背景**

辣椒是全球范围内比较重要的农作物，主要有5个栽培种：一年生辣椒、小米椒、黄灯笼辣椒、风铃辣椒和茸毛辣椒。这五大栽培种辣椒均有12条染色体。一年生辣椒栽培品种Zunla‐1和CM334在2014年完成测序。风铃辣椒栽培品种PBC81和黄灯笼辣椒栽培品种PI159236已经也已经完成测序。这些研究显示辣椒不同栽培种基因组大小的变异在3.2～3.9 Gb之间，其中约有80%的基因组区域为重复序列。为了研究种内遗传变异与多样性，进来的一些研究团队对一些重要的的作物品种开展了泛基因组研究，其中包括玉米、大豆、水稻及小麦。这些物种的泛基因组研究显示物种内的基因存在-缺失变异（presence-absence variation；PAV）影响了不同栽培种之间的性状差异。

**Abstract**

该文章报道了第一个辣椒的泛基因组研究。辣椒基因组的高度重复特性导致单核苷酸多态性SNP的鉴定存在潜在的错误，因此作者在进化分析和全基因组关联分析时并未基于SNP的数据，而是采用了PAV的结果。利用基因上read的覆盖度信息，作者还鉴定了辣椒素和类胡萝卜素这两个辣椒区别于其它作物最重要的化合物的生物合成通路关键基因的遗传变异。作者还构建了一个在线网站以方便全球的研究者获取有关辣椒泛基因组的数据。


**Results**


常规的利用de novo assembly的方法从383株辣椒中，建立了辣椒的泛基因组，然后用Maker进行注释，筛选出高质量的被注释的基因。

然后利用sequence-read寻找基因的PAV（参数是CDS coverage 大于0.6 和Gene coverage 大于 0.5）。根据基因的PAV画系统进化树，树种有4个cluster，如图a。作者将pangenome的基因分为了十五类，根据他们在Venn的集聚情况，是不是某种辣椒特有的基因。然后研究者也建立了一个网站来展示相应的信息。

然后是本文的亮点GWAS-PAV，gene pan06g005570有明显与traits关联的信号。然后他们发现25 株C. annuum 在pan06g005570有明显的缺失，然后都是产生了黄色或者橙色的果实。然后他们也发现了，C. annuum 中辣椒素的株系会在 pan02g021380附近有缺失，(Pun1) region区域。

**小结**

这篇文章由于是发在letter上，所以方法上很多信息不是很全面，要是方法更全面会很有兴趣去学习他如何做GWAS和PAV的研究。

另外这篇文章也侧面说明了，现在泛基因组想发好的journal，必须有GWAS-SNP或者SV/PAV，加数据库的建立才行。




![][2]


  [1]: https://nph.onlinelibrary.wiley.com/doi/full/10.1111/nph.15413
  [2]: https://wol-prod-cdn.literatumonline.com/cms/attachment/717f69a4-5b7a-4470-810b-4acfd603e39e/nph15413-fig-0001-m.jpg
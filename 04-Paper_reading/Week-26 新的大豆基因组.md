# Week-26 新的大豆基因组

标签（空格分隔）： 文献解读

---

**Titile:**
De novo assembly of a Chinese soybean genome

[文章链接][1]


**Abstract:**


大豆在中国被驯化，已成为最重要的油籽作物之一。由于其引入和传播的瓶颈，来自不同地理区域的大豆表现出广泛的遗传多样性。亚洲是最大的大豆市场;因此，来自该地区的高质量大豆参考基因组对于大豆研究和育种至关重要。在这里，研究者通过结合SMRT，Hi-C和光学测绘数据报告中国大豆基因组的“中黄13”的从头组装和序列分析。组装的基因组大小为1.025Gb，contigsN50为3.46Mb，scaffoldN50为51.87Mb。该基因组与先前报道的参考基因组（cv.Williams 82）之间的比较揭示了超过250,000个结构变异。共有52,051个蛋白质编码基因和36,429个转座因子被注释用于该基因组，并且还建立了包括39,967个基因的基因共表达网络。这种高质量的中国大豆基因组及其序列分析将为未来大豆改良提供有价值的信息。


**Results：**

**Genome sequencing, assembly and annotation**

![][2]

不同技术所做出来的assemblies的比较，运用所用技术糅合在一起的组装(pacbio+bionano+hi-c）具有最好的组装。Busco的结果再次验证。

常规的Repeat 还有TE 的分析。然后这组装还组装除了叶绿体的基因

![下载 (1).png-54.1kB][3]


**Genome comparison to the Williams 82 reference genome**

新做出来的基因组与旧的基因组的相互比较。发现了一些列的结构变异(translocations, inversions and presence variations).试图将PV和开花颜色表型结合。

![此处输入图片的描述][4]




**Gene co-expression network assists mining of important agronomic genes**


基因共表达网络是探索基因调控关系的流行方法。同一模块中的基因具有相似的表达模式和趋向于相同的生物学功能。基因共表达网络可用于预测基因功能和鉴定生物途径中的关键基因。图形高斯模型（GGM）采用部分相关系数来测量基因之间的直接相关性，是一种用于共表达网络分析的稳健方法，并且之前已经为拟南芥和玉米建立了GGM基因网络。为了促进Gmax_ZH13基因组的实际应用，研究者使用来自NCBI SRA中存储的1,978个大豆RNA-seq运输的转录组数据集，基于来自Gmax_ZH13的基因构建了大豆GGM基因共表达网络。不同发育阶段的25种不同组织。已建立的Gmax_ZH13 GGM网络包含39,967（76.78％）个基因和330,864个共表达的基因对。


数量性状基因座（QTL）和全基因组关联研究（GWAS）通常用于探索控制农艺性状的基因，但它们通常会导致大的候选区域，使得难以挖掘因果基因。 QTL / GWAS和基因共表达网络技术的组合可以帮助鉴定特定QTL / GWAS区域中的因果基因。研究者通过探索控制大豆开花时间和亚油酸含量的新基因来测试这种方法。


这个部分相对来说是个亮点，其方法可以借鉴并且用于其他的研究。

 - 从NCBI SRA下载RNA-sq数据
 - trim 处理RNA-seq数据，使用STAR进行mapping
 - 去除低比对率的RNA-seq lib 还有小的RNA-seq lib
 - 使用RSEM进行表达量统计
 - 去除低表达量的基因 <10 runs, expression values (TPM) ≥5,  assembled into a gene expression matrix with 42,169 rows (genes) and 1,978 columns (runs)。
 - 矩阵用于通过随机采样方法进行的部分相关系数（pcor）计算  **GeneNet package in R** 
 - 计算了基因之间的Pearson相关系数（r）
 - 330,864 gene pairs with pcor≥0.035 and r≥0.35 were selected for GGM gene co-expression 用于共表达基因对网络的构建。



 
![此处输入图片的描述][5]


  [1]: http://engine.scichina.com/publisher/scp/journal/SCLS/doi/10.1007/s11427-018-9360-0?slug=full%20text
  [2]: http://static.zybuluo.com/lakesea/2j85550y8865910bw9dxa19f/%E4%B8%8B%E8%BD%BD.png
  [3]: http://static.zybuluo.com/lakesea/ejkeju765256cnsrwv5a7wz7/%E4%B8%8B%E8%BD%BD%20%281%29.png
  [4]: http://engine.scichina.com/cfs/files/figures/PrBpgTguSqYE9Njve/scls-2018-0206-t1.png
  [5]: http://engine.scichina.com/cfs/files/figures/cxfi2H8JanzCdsC3v/scls-2018-0206-t2.png
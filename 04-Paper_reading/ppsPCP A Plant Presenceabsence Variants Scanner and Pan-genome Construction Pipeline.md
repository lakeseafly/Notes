# ppsPCP: A Plant Presence/absence Variants Scanner and Pan-genome Construction Pipeline

标签（空格分隔）： 文献阅读

---

供稿人：lakeseafly

文章信息
题目：ppsPCP: A Plant Presence/absence Variants Scanner and Pan-genome Construction Pipeline
杂志：Bioinformatics
时间：10/03/2019
链接：https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/btz168/5372683

figure

![][1]

**文章介绍：**

**简介**

在原核生物的泛基因组研究中早早已经有很多工具被设计并使用。对于植物来说，由于其基因组比较复杂，比较大，一直没有很完善的流程被设计出来。该文章讲述了一个最新的(其实就是总结了一下之前泛基因组文章的内容)植物泛基因组流程ppsPCP。它能够扫描存在/缺失变异（PAV）并构建完全注释的泛基因组。研究者相信ppsPCP将有助于植物泛基因组研究，并帮助研究人员研究遗传/表型变异和基因组多样性。

**流程概述**
主要步骤分为十步:

 1. 全基因组de novo assembly，然后每个材料的全基因组和参考基因组比对去找出新的独特的non-references的序列。这一步会使用到MUMmer。
 2. 比对的结果回用于扫描PAVs，最短的PAV长度是被定义为100bp。
 3. 为了确认这些PAV，BLASTn用于在参考基因组上搜索这些PAV序列，确认他们是存在/缺失。
 4. 将上面BLASTn的结果用来判断PAVs序列可不可信，分别有两种情况:1)与参考序列非常接近(相似度>=95% 和该片段>=90%的序列被覆盖到)，这种情况下,这些高度相似的片段会被过滤掉。2)PAVs序列在参考基因组中找不到。另一句话说他们是缺失的。要被选出来。
 5. 这些被选出来的PAVs，进一步和query片段比较，如果他们是重叠的，就进而延长他们。
 6. 所有被挑选出来并延长的PAVS片段，会挪到一起，每个片段之间使用100bp的N-bases相连.然后准备进行注释
 7. 最后这些新的PAVS片段和参考基因组就组成了泛基因组
 8. 每个材料的基因组和参考基因组比对使用BLAT工具。
 9. 提取的基因区域是从query基因组和新生成二代PAV基因组中挖掘出来的（第五步中的结果）
 10. 一个完整的基于参考基因组的泛基因组，是通过合并PAVS序列文件及其与参考基因组的注释来构成泛基因组的

**个人评价**
总的来说该工具很好的总结了目前比较流行的建立泛基因组，探索PAVS的方法。和我们组的方法对比在不同的步骤有着一些差异。下一步，我打算比较一下我们组的方法和该文章的方法，看看两种方法做出来的泛基因组还有PAVs的片段有什么不同。


  [1]: http://static.zybuluo.com/lakesea/n6ss69n4igzp9qcq5t1zt00a/12103451_16eda5cb2c.png
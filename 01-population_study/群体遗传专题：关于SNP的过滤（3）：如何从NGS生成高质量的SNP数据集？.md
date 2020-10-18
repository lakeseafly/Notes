# 群体遗传专题：关于SNP的过滤（3）：如何从NGS生成高质量的SNP数据集？

标签（空格分隔）： 群体遗传

---

> 这周的推文是关于变异过滤的最后一节，主要讲讲如何生成高质量的变异数据集，还有讨论一些常用的过滤参数。


###背景知识

随着高通量测序方法的出现，大量DNA序列可以应用于SNP的发现。这些数据中SNP的识别依赖于计算方法。它们取决于您是否对种系或体细胞突变感兴趣。往往它们通常将序列与参考基因组比对，并根据你设置的参数进行过滤。

然而，利用NGS来 call SNP不是一件小事。获得的序列通常包含涉及不确定性，偏差和讨厌的假阳性结果的细微但复杂的错误。这意味着必须获得高质量的NGS数据，这需要精确的文库制备（包括DNA大小选择）来获得。完成所有操作后，以下是生成高质量SNP数据集的主要步骤：

###变异的calling还有基因分型

在第一步中，你必须确定至少一个样本的序列与参考序列有存在不同的位点。下一步，评估所有变异位点的单个等位基因，这一步又叫“基因分型”。大量的生物信息学软件可用于帮助您在NGS的序列中找到变体。对于单样本变体calling，您可以使用Atlas-SNP2，SLIDERII和SOAPsnp。对于基因分型或多样本变体calling，FreeBayes，GATK，QCALL，SAMtools和SeqEM。

处理信息的方式取决于拥有的序列覆盖类型。测序覆盖率是指在测序实验期间reads单个碱基的平均次数，可以用以下公式表示：

**Coverage = Read count x Read length / Target sequence size**


例如，10x覆盖率意味着每个碱基已被10个序列读取，而100x覆盖率意味着每个碱基已被100个序列读取。对碱进行测序的频率越高，读数的覆盖率越高，可靠性也越高。


使用高覆盖率测序数据，可以简单地忽略低质量等位基因，并仅计算遇到的高质量等位基因。另一方面，如果你正在处理中低覆盖率测序数据，你可能需要使用概率或贝叶斯方法来避免杂合子基因型的调查。在确定给定基因座是杂合的还是纯合的之前，需要考虑额外的信息，这包括读取覆盖率，NGS平台的错误率和对齐质量分数等信息。

###通过Joint Calling 来增加你的灵敏度

接下来，要取出假阳性的变异calling结果，首先你需要提高你calling 的特异性。一般来说有两种方法：


第一个策略是硬过滤。此策略假设假阳性，通常显示不寻常的特征，例如：

 1. reads的重复区域的低质量分数导致比对很模糊
 2. 由不完整的参考序列引起的与参考序列的效果很差的比对
 3. 链偏移（通常，真正的变体将在正向和反向链上具有相同的覆盖率。但是，在这种情况下，变体仅由正向链或反向链一条链支持）
 4. 在特定区域内聚集的大量变体

在硬过滤中，将通过为这些变体特征设置特定条件并最终排除不符合标准的变体来消除这些问题。然而，由于覆盖不均匀（人类主要组织相容性复合物或白细胞抗原基因）和重复区域（端粒或着丝粒），硬过滤通常是棘手的。这种类型的过滤常会用于GATK VariantFiltration和VCFtools中。
 
关于这个硬过滤的标准，其实没有一个完美的标准，不同人不同数据不同文章都会使用不同的参数，下面简单介绍一些常用的硬过滤的标准：

####GATK论坛官方推荐

**For SNPs:**

    

    QD < 2.0
    MQ < 40.0
    FS > 60.0
    SOR > 3.0
    MQRankSum < -12.5
    ReadPosRankSum < -8.0


**For indels:**

    QD < 2.0
    ReadPosRankSum < -20.0
    InbreedingCoeff < -0.8
    FS > 200.0
    SOR > 10.0

一般会再上面的基础加一个QUAL < 30.0，有些公司使用的参数则是QUAL<30.0 ||QD < 2.0 || FS > 60.0 || MQ < 40.0，另外在全基因组测序数据中，通常会根据基因覆盖度增加测序深度的筛选条件。比如DP>10。

**网上比较多人总结使用的标准：**

    MinDP (Minimum read depth最小read的深度):   5 (Indels) and 3 (SNPs)
    
    MaxDP (Maximum read depth最大read的深度):  如果你的数据是覆盖率比较低的，一般将它设定为 100. 其他一般的数据将它设为平均覆盖度的三倍.
    
    BaseQualBias (Minimum p-value for baseQ bias):  0
    MinMQ (Minimum RMS mapping quality for SNPs):  20 or 30 (to be more stringent)
    
    Qual (Minimum value of QUAL field):  15 or 20
    
    StrandBias (Minimum p-value for strand bias):  0.0001
    
    EndDistBias (Minimum p-value for end distance bias):  0.0001
    
    MapQualBias (Minimum p-value for mapQ bias):  0
    
    VBD (Minimum Variant Distance Bias):  0 (More relevant to RNA-seq reads)
    
    GapWin (Window size for filtering adjacent gaps):  30 bp
    
    SnpGap (SNP within INT bp around a gap to be filtered):   20 bp
    
    SNPcluster (number of snps within a region): 通常如果在10bp的序列中含有三个snp，一般会将这种不正常的snp直接去掉

其实最好的硬过滤方法是根据你自身数据来制定相应的阀值，换句话说就是将你call 出snp的阀值进行统计，然后将其统计绘制成正态分布图，然后再根据自身的情况截取前百分之10% 或者25% （百分比根据实际情况而定）。
 
第二种策略使用软过滤，在高覆盖率数据集的情况下生成可靠的结果。该策略本身也使用机器学习方法来解决覆盖率较低的特异性问题。然而，这个策略并非没有缺陷。该策略的一个主要缺点是需要从以前的研究中获得大量高质量变体的数据集。因此，它目前仅限于已经拥有广泛且易于获得的变体数据库的动物研究。 GATK VQRS软件可以帮助实现此策略。

###Double Check 变异的质量

在整个过程中仔细检查变体质量是非常的重要的。通过目视检查或使用独立技术验证您的实验来实现。对于可视化的检查，可以使用IGV，GenomeView和SAVANT等软件。还应该使用简单的计算来评估数据集的整体质量。


首先，观察到的SNP数量应该在物种多样性的估计范围内。接下来，SNP的数量应该与染色体物理长度相关。最后，纯合子和杂合子基因型的比例不应违反Hardy-Weinberg定律，这意味着携带SNP的每个二倍体个体在SNP基因座处应该表现出1.0的纯合子或0.5的杂合个体的等位基因频率。目前的就差不多了，我在本文中可能遗漏了一些其他的问题，也欢迎大家提问留言。
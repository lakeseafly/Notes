# 群体遗传学习笔记：NGS结构变异检测原理



标签（空格分隔）： 群体遗传

---


随着测序的成本越来越低，测序技术越发先进，除了研究单核苷酸多态性(SNP)。研究者们开始慢慢将目光转向了各位复杂，但是也同样非常常见的结构变异(SV)。接下来几期推文会和大家一起学习结构变异的检测原理，然后通过一系列的实战演练和大家一起熟悉一些相关的工具和流程。

## 结构变异的定义与其研究的重要性

变异是导致基因组差异的最重要因素，具体可分为单个碱基对的变异（SNVs/SNPs）、小的插入或缺失（InDels≤50bp）以及结构变异(SVs>50bp)。

根据结构变异的不同类型，结构变异可以进一步分为DNA序列的**插入、缺失、重复、倒位、易位、拷贝数变异**等。结构变异可以影响基因组的稳定性、相关基因的表达调控，进而决定物种表型。据统计，每个人类基因组都有超过20000个结构变异，**基因组结构变异可能导致的疾病**已经超过1000种，例如我们熟悉的耳闻的渐冻症、精神分裂症以及癌症。**结构变异带来的影响比SNVs/SNPs或者是InDels带来的影响更大**。另外，稀有且相同的一些结构性变异往往和疾病（包括癌症）的发生相互关联甚至还是其直接的致病诱因。结构变异除了广泛存在人类基因组外，也普遍存在于动植物之中。研究者发现，**结构变异与一些关键的农艺性状或者育种相关的性状也是息息相关**。并且多个研究表明，结构变异能够更好的解析群体结构，为我们更深入了解动植物驯化过程提供进一步的有效信息。简而言之，结构变异的相关研究必将是生物学未来的一个热点。

![][1]

## 结构变异检测原理

目前经典的结构变异检测方法包括：

 - Read Pair，一般称为Pair-End Mapping，简称RP；
 - Split Read，分裂read，简称SR；
 - Read Depth，简称RD，也有人将其称为RC——Read Count的意思，它与Read Depth是同一回事，都是利用序列read的横纵覆盖情况来检测变异是否存在的方法；
 - 从头组装（de novo Assembly， 简称AS）的方法。

![][2]
 
### Read Pair方法

简单来说，Read Pair方法是根据pair-end两端之间距离（插入片段）与参考基因组上差异来确认结构变异。根据测序的原理，这个插入片段长度是一个固定的值，通过测量插入片段的分布是RP方法进行变异检测的一个关键点。

RP方法共有两种策略来鉴定SVs/CNVs：

 - clustering approach：使用预先定义的距离（predefined distance）来检测异常（discordant）的PE reads
 - model-based approach：对全基因范围的PE read的插入长度进行概率分布建模，然后基于统计检验得到异常的PE reads；


下面引用黄树嘉老师（公众号解螺旋的矿工中一文）所写的一段话来解析一下RP探测结构变异的具体原理（我想不出我还能比他解析得更好）：

> **如果插入片段长度有异常，它实际上包含的意思是，组成read1和read2的这个序列片段和参考基因组相比存在着序列上的变异**。举个例子，如果我们发现它这个计算出来的插入片段长度与正态分布的中心相比大了200bp（假设这个200bp已经大于3个标准差了），那么就意味着参考基因组比read1和read2所在的片段要长200bp，通过类似这样的方式，我们就可以发现read1和read2所在的序列片段相比与参考基因组而言发生了200bp的删除（Deletion）。
> 
> RP除了可以利用异常插入片段长度的信息进行线性变异（特指Deletion和Insertion）的发现之外，通过比对read1和read2之间的序列位置关系，**还能够发现更多非线性的序列变异。比如，序列倒置（Inversion）**，因为，按照PE的测序原理，read1和read2与参考基因组相比对，正好是一正一负，要么是read1比上正链，read2比上负链，要么是反过来，而且read1和read2都应处于同一个染色体上，如果不是这种现象，那么就很可能是序列的非线性结构性变异所致，比如前者是序列倒置（Inversion），后者是序列易位（Translocation）等。

![][3]

接着简单总结一下利用RP这个方法去探测结构变异的流程。通过PE reads的比对找出那些**比对不正常**的read pairs，进而提取出比对不正常的区域，发现相互关联的聚集。根据这些区域的位置，大小，还有比对不正常的reads的数目来判断对应的结果变异的种类，最后通过计算相关的置信值来确定对应的结果变异。

使用RP方法来检测结果变异的不足：

 1. 由于RP方法主要是基于入片段长度的变化来检测Deletion。RP对于比较长的Deletion（通常是大于1kbp）比较敏感，准确性也高。但是对于小片段的Deletion，例如小于200bp的片段，其检测可靠性准确性就会降低。
 2. 它所能检测的插入Insertion序列，长度无法超过插入片段的长度。如果其长度超过插入片段，相当于是整个片段都是插入序列，PE read将无法比对到基因组上，从而无法进行检测。

Delly和Breakdancer是两个最经典的，使用RP方法来进行结构变异检测的软件。


### Split Read方法

Split Read方法又叫分裂read方法，其算法核心与RP一样，都是利用PE read来进行变异检测。但是正如它的名字一般（**分裂**），SR方法对应着一个特殊的情况：两条PE的read，有一条能够正常比对上参考基因组，但是另一条却不行的情形。其能够检测的结构变异类型包括：

![][4]

关于SR的优缺点，继续引用黄老师的原话（不是我懒，是人家写的真的很好）：
> **SR的一个优势在于，它所检测到的SVs断点能精确到单个碱基，但是也和大多数的RP方法一样，无法解决复杂结构性变异的情形**。对于SR来说，它要求测序的read要更长才能体现它的优势，read太短，许多变异都会不可避免地被漏掉，它的检测功效在基因组的重复区域也会比较差。

Pindel，Delly，lumpy和SVseq2都是使用SR方法检测结果变异的经典工具。

### Read Depth方法

Read Depth方法，主要通过在指定区域内（根据滑动窗口）的序列read的横纵覆盖情况来检测变异是否存在的方法。该方法目前普遍被使用于基因组拷贝数变异检测(CNV)。CNV实质上是序列Deletion或Duplication。Deletion就CNV中最极端的情况，相当于没有任何copy。这种极端的变异也可以被叫做存在或者缺失变异(PAV)。

全基因组测序（WGS）得到的覆盖深度呈现出来的是一个泊松分布——因为基因组上任意一个位点被测到的几率都是很低的——是一个小概率事件，在很大量的测序read条件下，其覆盖就会呈现一个泊松分布，如下图：

![][5]

拷贝数增加会使得该区域的Read Depth高于期望值，而拷贝数缺失使得该区域的Read Depth低于高于期望值。根据滑动窗口读段深度来指示拷贝数扩增与缺失。在众多的软件中，CNVnator采取了RD的方法，目前已经被广泛地被用于检测CNV。

### AS方法


AS方法（从头组装），通过将不同个体的read从头组装成基因组，然后两两之间一起比较找出其中的差异。这应该是最经典，最直接有效找结构变异的办法。

![][6]

但这里，与上面的方法一样。局限于短序列，你组装的基因组可能会不完整片段化，导致无法进行精准的基因组比较。另外我们上面提到的所有方法其实都受限于read太短。由于read太短，不能在比对的时候横跨基因组重复区域；并且无法抓捕很多大的Insertion序列。为了避免测序序列短的缺点，我们可以通过三代长read测序来克服二代数据短带来的不便。但是长序列的引入，又需要考虑其错误高和测序价格相对较高的影响。**在最理想的情况下，基于三代测序的从头组装应该是基因组结构性变异检测上最有效的方法，它能够检测并且覆盖所有类型的结构性变异。**

### 总结

在最理想的状态，不考虑成本，并且组装的基因组的复杂性，基于三代测序的从头组装是最有效的结构变异检测方法。在剩下三个方法中，每个方法都有自己的优缺点，并没有说哪个方法会比其它方法有压倒性的优势。下表是不同结构变异检测所推荐使用的工具：


![][7]
 
 目前大家最常用的方法就是，将几个方法/工具结合在一起，取不同结果的重合区域作为最终的结构变异结构。进而减少由于二代短测序产生的假阳性结果。下一次内容会和大家进行一些相关的实战，敬请大家关注。
 
 
 参考资料：
 
 

 - [【简书】一篇文章说清楚基因组结构性变异检测的方法][8]
 - [学术前沿 | NGS方法进行拷贝数变异检测概述][9]
 - [明哥的GitHub：基于NGS SVs检测原理][10]

`

  [1]: http://static.zybuluo.com/lakesea/fuu6kd8puc6l136rzcijrkny/2-Figure1-1.png
  [2]: http://static.zybuluo.com/lakesea/1yr9wf2n43532ikotid96yy9/StructralVariation-SVs-detection-principle-outline.png
  [3]: http://static.zybuluo.com/lakesea/1ut54jkdk4fyd5dh6ho3smya/StructralVariation-SVs-detection-principle-ReadPair-1.png
  [4]: http://static.zybuluo.com/lakesea/vkr9aljd83tlzfkl6f5366y8/StructralVariation-SVs-detection-principle-SplitRead.png
  [5]: http://static.zybuluo.com/lakesea/w53wi5vw008h2lqrudz7wauw/StructralVariation-SVs-detection-principle-ReadDepth-poission.png
  [6]: http://static.zybuluo.com/lakesea/hs1lmelbx4k84pidanxhexmw/image.png
  [7]: http://static.zybuluo.com/lakesea/e4k7pfaz155k6abtawk1arnn/image.png
  [8]: https://www.jianshu.com/p/4c8e109f0e6a
  [9]: https://zhuanlan.zhihu.com/p/31529899
  [10]: https://github.com/Ming-Lian/NGS-analysis/blob/master/Structural-Variation.md#focus-on-cnv-discovery
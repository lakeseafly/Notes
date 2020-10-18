# 认识与了解主成分析PCA

标签（空格分隔）： PCA

---

###PCA概念介绍

PCA 全称是Principal Component Analysis，又叫**做主成分析**。是一种分析、简化数据集的技术。主成分分析经常用于减少数据集的维数，同时保持数据集中的对方差贡献最大的特征。这是通过保留低阶主成分，忽略高阶主成分做到的。这样低阶成分往往能够保留住数据的最重要方面。

主成分分析由卡尔·皮尔逊于1901年发明，用于分析数据及建立数理模型。其方法主要是通过对协方差矩阵进行特征分解，以得出数据的主成分（即特征向量）与它们的权值。PCA是最简单的以特征量分析多元统计分布的方法。其结果可以理解为对原数据中的方差做出解释：哪一个方向上的数据值对方差的影响最大？换而言之，PCA提供了一种降低数据维度的有效办法。

PCA基本原则在最大程度反映原变量所代表的信息，同时保证新变量之间信息不重复。在生物学上，常常用于将SNP信息浓缩为几个新的变量。一般而言，PCA分析之后会给出一个PCA图，而这个图往往会显示出某种分群的特性,这就是我们群体遗传中使用PCA图的原因，可以将其分群或者验证前面系统发育进化树和后面要讲到的structure图的结果。

###主成分析的内在算法

PCA听起来好像很强大，但具体其算法是怎样的？

以小鼠的RNA-seq数据为例，先看2个genes的表达量：

PCA的算法如下：

 1. 寻找中心点，并平移至中心点；
 2. 计算PC1；
 3. PC1是一条过原点的最佳拟合曲线，所有数据点与其的距离平方和最小。
 4. 计算PC2；
 5. PC2与PC1相垂直。

以PC1和PC2为标准坐标轴，绘制PCA图。

下面分别拆解上述步骤：

**寻找并平移中心点**

如下图所示，先对Gene1求平均值，在对Gene2求平均值，于是就可以获得所有数据的中心点，然后将坐标系原点平移至此中心点。

![][1]

**计算PC1**


计算PC1就是为了获得一条过原点的直线，所有数据点距离此直线的距离平方和最小。以一个点为例，也就是下图中1图的距离a最小，由于每个数据点距离原点的距离是固定的，所以使得距离a最小，也就是使得距离b最大。


最终可以求得最佳拟合直线如下图3所示，假定此直线斜率为0.25，也就是说"PC1由4份Gene1和1份Gene2构成"，计算方法是“奇异值分解”。

由于奇异值分解时，斜边c是标准化为1的，所以标准化之后，PC1由”0.97份Gene1和0.242份Gene2构成"


![][2]



**计算PC2**

在PCA中，各个主成分之间是相互垂直的，在这里PC2和PC1是垂直的，也就是PC2的斜率为4，经过标准化之后，"PC2由-0.242份Gene1和0.97份Gene2构成"。

于是PC1和PC2都已经找到，分别是下图中的红色虚线和蓝色虚线坐标轴

![][3]


**绘制PCA图**

![][4]
![][5]

于是所有的数据点都可以转换为坐标（PC1，PC2），以PC1和PC2为坐标轴即可绘制出相应的PCA图。

![][6]

2个Gene的PCA过程如上，3个Gene的PCA同理，先获得中心点，然后对所有点进行拟合获得PC1，然后在垂直于PC2的线中，找到最佳拟合线，最后PC3是垂直于PC1和PC2的。有几个样本就会有几个主成分，且主成分的重要性依次下降，PC1>PC2>PC3。



###主成分析的功能

**1. 简化运算**

在问题研究中，为了全面系统地分析问题，我们通常会收集众多的影响因素也就是众多的变量。这样会使得研究更丰富，通常也会带来较多的冗余数据和复杂的计算量。

例如，我们测序了100种样品的基因表达谱借以通过分子表达水平的差异对这100种样品进行分类。在这个问题中，研究的变量就是不同的基因。每个基因的表达都可以在一定程度上反应样品之间的差异，但某些基因之间却有着调控、协同或拮抗的关系，表现为它们的表达值存在一些相关性，这就造成了统计数据所反映的信息存在一定程度的冗余。另外假如某些基因如持家基因在所有样本中表达都一样，它们对于解释样本的差异也没有意义。这么多的变量在后续统计分析中会增大运算量和计算复杂度，应用PCA就可以在尽量多的保持变量所包含的信息又能维持尽量少的变量数目，帮助简化运算和结果解释。

**2. 降噪去除outliners**

比如说我们在样品的制备过程中，由于不完全一致的操作，导致样品的状态有细微的改变，从而造成一些持家基因也发生了相应的变化，但变化幅度远小于核心基因(一般认为噪音的方差小于信息的方差）。而PCA在降维的过程中滤去了这些变化幅度较小的噪音变化，增大了数据的信噪比。


**3. 进行多维数据可视化**

在上面的表达谱分析中，假如我们有1个基因，可以在线性层面对样本进行分类；如果我们有2个基因，可以在一个平面对样本进行分类；如果我们有3个基因，可以在一个立体空间对样本进行分类；如果有更多的基因，比如说n个，那么每个样品就是n维空间的一个点，则很难在图形上展示样品的分类关系。利用PCA分析，我们可以选取贡献最大的2个或3个主成分作为数据代表用以可视化。这比直接选取三个表达变化最大的基因更能反映样品之间的差异。（利用Pearson相关系数对样品进行聚类在样品数目比较少时是一个解决办法）

**4. 发现隐性相关变量**

我们在合并冗余原始变量得到主成分过程中，会发现某些原始变量对同一主成分有着相似的贡献，也就是说这些变量之间存在着某种相关性，为相关变量。同时也可以获得这些变量对主成分的贡献程度。对基因表达数据可以理解为发现了存在协同或拮抗关系的基因。






参考资料：

 - 一文看懂PCA主成分分析（易生信）
 - 一文看懂主成分分析 （生信技能树）
 - StatQuest生物统计学专题 - PCA （生信菜鸟团）


  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/iaRJcrq2LosicDnom8OhpWxPLKwZcnkKh9UuSBwzCoAXY9OdzibY3rWh2DouAAlHxrmuiaibfKR1YwJK88lbnqh98Bw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/iaRJcrq2LosicDnom8OhpWxPLKwZcnkKh9yBo5Z7svCCIVLMvpo7Z9gcBYNdkvjxaC41QPYCHAbZq8dukFkua0FQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [3]: https://mmbiz.qpic.cn/mmbiz_jpg/iaRJcrq2LosicDnom8OhpWxPLKwZcnkKh9uU4pCCyKayDrMyXVmMXPThIQ1ePIqr1Bw1OWFzQiaxy1ll4Knh9oP6Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [4]: https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2LosicDnom8OhpWxPLKwZcnkKh9I7ENe4ut8icKBp0fMOar8M5X0iar7jfgLDNqW3xqesdRicUtHYpA3XciaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [5]: https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2LosicDnom8OhpWxPLKwZcnkKh97lQ34ZF5PSU6Ej7l2ibuJQa6lGpZa5lqzFArxV9OhrQezOJicrB4JJ1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [6]: https://mmbiz.qpic.cn/mmbiz_jpg/iaRJcrq2LosicDnom8OhpWxPLKwZcnkKh99rFv7rZDiaZ6ctg12NHVAtJh9z9hDYkzg9cnlL8M7CiacMEnBI9YNeQw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
#Structure图的生成实战


标签（空格分隔）： 群体遗传学习小组

---

Structure分析当然最经典的软件就是STRUCTURE。但Structure分析还有其他软件可以选择: ADMIXTURE、FRAPPE。这两个软件的运行速度都大大超过STRUCTURE。但FRAPPE的不足没有提供方法估算最佳K值。ADMIXTURE使用与STRUCTURE相同的模型，而且运行效率也很好，所以是一个比较推荐的软件。还有一款软件叫做FASTstructure是Structure的一个升级版，其优点是能快速处理大批量的文件。

对于软件的选择，其实都是大同小异，算法都差不多，一般选速度比较快的进行structure分析与图的绘制。

###实战一：Admixture

Admixture 很容易下载和安装（已经编译好的），下载地址：http://www.genetics.ucla.edu/software/admixture/

1. admixture的输入文件为：PLINK (.bed), ordinary PLINK2.过滤SNP文件 (.ped), or EIGENSTRAT (.geno)

所以需要使用vcftools进行格式转换：

```
vcftools --vcf my.vcf --plink --out plink
```

2.过滤SNP文件

像之前的PCA，还有建树过程一样，适当的过滤可以减少软件的计算量：

```
plink --noweb --file plink --geno 0.1 --maf 0.05 --hwe 0.0001 --make-bed --out out

```


3. 使用admixture进行k值的筛选

这里使用admixture提供的测试数据来计算：

```
for K in 1 2 3 4 5; do admixture --cv hapmap3.bed $K | tee log${K}.out; done
```


提取K值：

```
grep -h CV log*.out
CV error (K=1): 0.55248
CV error (K=2): 0.48190
CV error (K=3): 0.47835
CV error (K=4): 0.48236
CV error (K=5): 0.48985
```

其中K=3时候，有最小的K值，选K=3进行structure图的绘制

```
tbl=read.table("hapmap3.3.Q")
barplot(t(as.matrix(tbl)), col=rainbow(3),xlab="Individual #", ylab="Ancestry", border=NA)
```



![][1]


这里的分层比较明显，但有时候由于样本的杂合群体之间分层不是很明显，很多文章就会将k=某个范围的structure图都画出来，然后再进行下一步的分析。


###大数据集的应用

上面只是简单说了一下admixture怎么用，但真正处理大的数据时候需要更多的一些操作。下面引用3010水稻泛基因组文章。简单分析一下如何在大的群体中画structure图的思路。

> First, ADMIXTURE was run on 30 random 100,000-SNP subsets of the core SNP set with k (the number of groups) ranging from 5 to 18, and k = 9 was chosen because it was the minimal value of k to separate all previously known groups (cA, cB, XI, GJ-trp, GJ-tmp and part of GJ-sbtrp). With k = 9, ADMIXTURE was then run again on the whole core SNP set nine times with varying random seeds; the Q-matrices were aligned using CLUMPP software53 and clustered on the basis of similarity.

由于3000水稻这篇文章的样本量很大，如果全部进行K值得分析可能几个月都不能跑完。这里他将它的SNP dataset 随机分成30份，每一份都含有100k的 的SNP，然后分别将这些分开的100k的SNP进行K值得检验。他们发现当K=9 时候，在所有30组的测试中，K值是最低的。然后他们使用K=9，对核心SNP进行计算，再进一步画图。

文章中随机抽取100k测K值，简化了其计算的时间与电脑资源的压力，这种思路值得我们去学习借鉴。


参考文章：

 1. Genomic variation in 3,010 diverse accessions of Asian cultivated rice
 2. https://www.jianshu.com/p/ef6350877105

  [1]: http://static.zybuluo.com/lakesea/2b81k7ug021mx8q05clinqry/Rplot01.png
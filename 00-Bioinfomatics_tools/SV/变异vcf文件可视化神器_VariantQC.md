# 变异vcf文件可视化神器:VariantQC

标签（空格分隔）： 生信工具

---

今天给大家带来一款非常方便好用的SNP变异vcf文件可视化神器:**VariantQC**。听到这个名字有没有似曾相识的感觉，嘿嘿嘿。没错，那就是FastqC了。相信大家对这个软件肯定再再再熟悉不过了，毕竟它就是每个人接触学习生物信息学一开始都会使用到的工具。相信通过这么一类比大家大概也知道**VariantQC**是拿来干嘛的了。好废话不多说，下面和大家学习一下该工具的使用，简单了解一下其方便好用的功能。


###工具简介

首先简单说说该工具的产生背景。大规模基因组研究往往会产生成百上千万个变异序列。为确保变异和基因型数据的一致性和准确性，每个研究者都有必要在进行下游分析之前评估变异文件的质量。然而产生的数据集由于太大，基本不可能通过人工进行检查检验。另外，变异文件VCF并不便于生成变异摘要统计信息。

在这重重困难之下，变异文件可视化神器**VariantQC**填补了这一空白。这个工具不但能为用户提供友好的交互式的变异可视化QC报告，生成并简明地总结了VCF文件的统计数据。更强大的是，该工具可以分别按照染色体，样本和过滤条件分别汇产生汇总报告(后面会给大家详尽介绍)。**VariantQC**产生的报告对于高级数据集摘要，质量控制和帮助标记具有异常值的变异位点或者样本非常有用。此外，VariantQC可以直接在VCF文件上运行，因此可以轻松地加载到不同变异calling的流程中。

###工具下载

**VariantQC**其实是DISCVRSeq其中一个工具集，是一个JAVA编写的程序。直接到GitHub上复制，就能直接使用了。

通过GitHub下载：
```
git clone https://github.com/BimberLab/DISCVRSeq/
```

帮助文档地址:
```
https://bimberlab.github.io/DISCVRSeq/
```

正如上面提到的，**VariantQC** 可以通过`DISVRseq.jar`进行调用：

```
java -jar DISCVRseq.jar VariantQC --help

java -jar DISCVRseq.jar VariantQC -O output_file -R ref.fa -V variant.vcf

###主要的三个参数分别是-O,-R和-V,分别对应着输入文件的名称，参考序列文件和变异vcf文件。

```

当然如果你有兴趣了解`DISCVRseq`中其它的工具，可以通过下面的命令进行查看:
```
java -jar DISCVRseq.jar --list 
```



###实战展示

在使用**VariantQC**之前，需要进行一些准备工作。首先是要分别对参考序列和变异vcf文件进行index:

对参考序列进行index，这一步一般在使用GATK去callSNP之前都会去做:

```
samtools faidx genome.fasta 
java -jar picard.jar CreateSequenceDictionary R=genome.fasta O=genome.dict
```

然后就是给变异文件进行index，生成`xxx.vcf.idx`文件,这里使用到GATK4.0:

```
java -jar ~/biosoft/gatk-4.1.3.0/gatk-package-4.1.3.0-local.jar IndexFeatureFile -F variants.vcf
```

准备工作做好了就可以开始使用我们的神器了,其运行时间和你VCF文件的大小，变异的数量有关，这里我使用的是一个简单的测试文件，不到一会就运行完了，生成了一个`.html`格式的报告文件:

```
java -jar DISCVRseq.jar VariantQC -O report.html -R ref.fa -V variant.vcf
```

那么事不宜迟，现在就让我打开潘多拉宝盒。使用你的浏览器打开生成的html格式文件,来看看这个报告长成什么样子:

![][1]


这里由于长度的现在，只截取了部分的图片。首先呢，该工具可以从四个角度来可视化VCF文件，分别是**Entire VCF,By Contig,By Sample** 和**By Filter Type**。我个人是特别喜欢**By Sample**，因为极大的方便我找到看到不同样本中VCF变异的数量等，以便快速发现outliner。

然后在每一个可视化的角度，里面提供了不同类别的VCF变异文件的统计数据，这里我截取**Entire VCF**角度下的统计数据，和大家简单了解一下其可视化的效果：

####Variant Summary
这个图中可以看到被called的，被filtered的变异(我的测试数据没有进行filter，所以这一行是空的)和原始Raw的变异统计信息。统计信息有多达27列，例如总的位点，被called的位点，总的变异位点数等等一些列的统计数据。你还可以使用上面的`Configure Columns`功能进行自定义你的列，然后使用`Plot`功能画出你想要的列的图。
![][2]

###Variant Type
然后就是变异的类型，这里可以按照变异所对应的reads的数目或者其对应的百分比进行展示。这样就是可以清晰看到各种SNPs，Insertions，Deletions一系列变异的数量。该图片也是可以使用`export plot` 功能随时输出的。


![][3]


####Genotype Summary

下面是基因型统计总结,展示了被called还有没有被called的基因型数目:

![][4]

####SNP/Indel Summary
接着是SNP和Indel的统计总结，包括了SNPs的数目，singleton SNPs的数目，Indels数目等等的信息。

![][5]

####Ti/Tv Data
这个是Transition 和 Transversion变异的总结： 
![][6]

最后两个是按照变异过滤的类型和进行统计的由于这里我没有进行进行过滤，其输出的图片和信息基本没什么意义，就不继续展示了。

另外这个报告右侧还有一个非常好用小工具栏：
![][7]
便于你将图表重命名，highlight你感兴趣的样本，隐藏你不想展示的样本，还有下载对应的图等等，一系列实用帮助你快速出图的小功能。


好啦，今天介绍就到这里了。纸上得来终觉浅，大家可以使用自己的数据测试了解一下。如果你没有合适的数据也可以到评论区的百度云链接上下载我的测试报告，拿来耍一耍。最后大家有没有觉得这个工具很溜，那么请转发，点击文末的好看和关注我们，素质三连发支持起来，分享给更多的朋友。

  [1]: http://static.zybuluo.com/lakesea/b52g8871hes7jqc25u5k6qle/image.png
  [2]: http://static.zybuluo.com/lakesea/4xzd08xmp9qpffqxyyukcif7/image.png
  [3]: http://static.zybuluo.com/lakesea/6adbitrehw7nt1oroz20eevw/image.png
  [4]: http://static.zybuluo.com/lakesea/i2ofsrraeukvs80ijc1egte1/%E6%8D%95%E8%8E%B7.PNG
  [5]: http://static.zybuluo.com/lakesea/x2szqioof6d0l5ht2byoi0dw/image.png
  [6]: http://static.zybuluo.com/lakesea/d2tbku7w9rgf7j61r9fkkb82/image.png
  [7]: http://static.zybuluo.com/lakesea/x8g5aqpnqm1sr9094fobf0ru/image.png
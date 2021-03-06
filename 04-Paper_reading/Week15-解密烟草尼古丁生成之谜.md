# Week15-解密烟草尼古丁生成之谜

**题目：Wild tobacco genomes reveal the evolution of nicotine biosynthesis**

**Abstract**
尼古丁是烟草中特有的，会使人上瘾的一种生物碱。可是，我们对其生物合成通路的研究还相当有限。研究者测序和组装了两种野生烟草Nicotiana attenuata (2.5 Gb) 和 Nicotiana obtusifolia (1.5 Gb)，这两种野生烟草是自然状态下研究适应性很好的生物模型。研究发现在茄科发生全基因组三倍化事件之后，烟草基因组整套的TEs(转座因子)发生了扩张，促进了发生复制的基因的表达多样性，从而促使取食诱导信号和防御信号的形成，这就包括尼古丁合成通路。这种根部合成尼古丁的生物合成机制，来源于两个古老的初级代谢通路（聚胺和辅酶通路）的逐步复制。NAD途径中的谱系特异性重复和茄科特异的、在根中表达的乙烯应答因子激活了整个尼古丁合成基因的表达，从TEs中衍生的转录因子结合物可能有助于尼古丁生物合成途径基因的共表达和代谢通量的协调。总之，这些结果证明TEs和基因复制促进了与植物适应性相关的重要新代谢作用的出现。


**Results**

**基因组组装注释**

N. attenuata基因组采用的是30× Illumina short reads、4.5× 454 reads 和10× PacBio single-molecule long reads，组装得到2.37 Gb基因组，并通过50×光学图谱和高密度遗传图谱将基因组组装到染色体水平，12条连锁群组装长度为825.8 Mb，contig N50长度为90.4 kb，scaffold N50长度为524.5 kb。同样的，采用50× Illumina sort reads组装得到N. obtusifolia基因组，contig N50长度为59.5 kb，scaffold N50长度为134.1 kb。对N. attenuata基因组进行基因预测，使用AUGUSTUS和MAKER2基因预测流程预测得到33,449个基因，其中71%具有RNA-seq reads支持，并且12,617个基因与拟南芥同源，18,176个基因与番茄基因同源。

**烟草三倍体化事件**
![此处输入图片的描述][1]
为了研究烟草的进化历史，通过与11个已发表物种进行系统发生分析，证实烟草与茄科物种共同经历了一次全基因组三倍化（WGT）事件。两种野生烟草基因组中至少有3,499对基因来源于WGT事件并在这两个物种中保留了下来，4dTV和TD分析表明这些基因中53.7%至少在一种组织中表现了表达变异，这就说明这些WGT事件衍生出来的重复基因形成了新的功能或者亚功能。


**转座子扩张**

![此处输入图片的描述][2]



根据“genomic shock”假说，多倍化事件的发生总是伴随着TE活动的爆发。野生烟草基因组中的LTRs明显高于其它茄科成员，N. attenuata高达81.0%，N. obtusifolia高达64.8%。TE插入历史分析发现，所有烟草的反转座子Gypsy在近期都发生过扩张，其中N. obtusifolia基因组较小，相比与其它烟草Gypsy拷贝数较低。最新研究指出辣椒在近期也经历过一次大规模的Gypsy扩张，时间推算是发生在WGT事件之后，稍微早于烟草的Gypsy扩张，这就说明茄科不同物种的Gypsy扩张是独立发生的。

除此之外，MITEs在进化中也非常重要。由于MITEs的位置长紧邻基因并且转录活跃，所以被认为能够促进基因调控的演化。N. attenuata基因组中共注释出13个MITE家族，其中几个家族在烟草中发生了扩张。其中茄科特有的亚家族DTT-NIC1是最丰富的，多富集于N. attenuata基因上游1 kb区域内，并且这些基因多为食草动物诱导的早期防御相关基因，这可能与引入防御相关基因的WRKY转录因子结合位点有关。

新陈代谢和信号通路网络构建的创新可能是复制事件后组织水平上基因表达模式变化造成的。通过对一次复制事件后的基因对表达变化及DTT-NIC1插入作用的分析发现，DTT-NIC1家族插入与基因表达变异和组织特异性显著相关，这就说明TE家族是烟草全基因组复制事件后基因调控变化的决定性因素。

**尼古丁有机合成的机理**
![此处输入图片的描述][3]
为了更好的理解基因复制和TE插入在烟草适应性进化历程中的地位，研究人员重新构建了尼古丁生物合成通路的进化历史。尼古丁的合成局限于烟草的根部，包括吡啶环和吡咯烷环的合成，并通过A622蛋白和BBL作用最终合成尼古丁。分析发现，吡啶环和吡咯烷环的合成来源于植物两大主要的代谢通路：烟酰胺腺嘌呤二核苷酸辅因子和多胺代谢通路。然而，这两种代谢通路的复制事件和模式并不相同，从而影响烟草生物碱代谢通路基因的差异。

N. attenuata不同组织的转录组分析表明，尼古丁合成相关基因都在根中特异性表达，并且在相应食草动物的茉莉酸信号通路中表达上调。实验已经证实乙烯响应因子亚家族IX和MYC2在上调的尼古丁基因中具有重要作用。MYC2的复制发生于茄科的全基因组多倍化和片段复制，其中有两个拷贝在烟草和几种茄科植物中保留了下来。

**总结**

通过对两个高质量野生烟草基因组的分析，为全基因组结构变异分析和尼古丁合成机制的演化提供了深入的基因组学研究。该研究阐明特有代谢物合成通路的形成和复杂适应性的进化历程中，基因复制和转座因子插入之间的互作具有重要的作用。



  
  


  [1]: http://mmbiz.qpic.cn/mmbiz_png/ENH69O0PkXMEhOugUmvH0RCiapNGWOicNJTKXsvwCZYrrZagzm4jeebbtAx0IdmBN6ia0BwE999p424DfPzF3gMpQ/0?wx_fmt=png
  [2]: http://mmbiz.qpic.cn/mmbiz_jpg/ENH69O0PkXMEhOugUmvH0RCiapNGWOicNJw2pQXR7OrbRJmxiasrGVplrdib59pibKvUQ1rewnLdjvs1W5f3pfJHkWA/0?wx_fmt=jpeg
  [3]: http://mmbiz.qpic.cn/mmbiz_jpg/ENH69O0PkXMEhOugUmvH0RCiapNGWOicNJ05Hy0dlkveSSwGXEvaIQAwicDnGRbbWE22U8Hia8I80z89qxrsU6ZDxA/0?wx_fmt=jpeg
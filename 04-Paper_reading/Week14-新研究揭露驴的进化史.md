# Week14-新研究揭露驴的进化史


**题目：Improved de novo genomic assembly for the domestic donkey**      [Link][1]


**Abstract**

驴和马共有一个可追溯到大约4百万年前的共同祖先。尽管马的染色体水平上有高质量的基因组组装，但目前用于驴的组装仅限于scaffold level。由于没有更好的驴组装质量，阻碍了涉及在全基因组范围内表征遗传变异模式的研究。研究者使用Hi-C组装技术获得的新的高质量驴基因组组装技术，提供次染色体level的组装。并利用这个新组件来获得更准确的马群以外的马类物种，包括全基因组和本地物种的杂合度测量，以及检测可能与家养驴中正选择有关的纯合子运行。最后，这个新的组装使我们能够确定马和驴之间的细微染色体重排，这可能在它们的分化和最终物种形成中起到积极作用。


**Background**

**什么是Hi-C？**

Chicago属于体外的Hi-C建库技术，提取DNA后，通过人工模拟构造一定长度的染色质，然后进行甲醛交联，酶切，标记，片段筛选，构建Chicago文库进行测序，以 HiRise流程进行组装。

组装原理与体内Hi-C相同：同一条染色体内的互作频率远高于染色体间的互作；同一条染色体内部，两点间距离越远，互作强度越低。利用此特征可将原始contigs聚类、排序、定向，组装至染色体水平。现在也越来越多基因组用到Hi-C技术来进一步提升至染色体水平。


**Results**


**组装注释结果**

不用说，对比以往旧的组装结果本文新的组装结果有了进一步的提升。N50：15.4Mb vs 3.8Mb, 差不多12Mb的提升，scaffold的长度也有明显差别。但是值得注意的一点是，基因组预测得到18,984个蛋白质编码基因，与旧的驴基因组，还有马的基因组相比其蛋白质编码基因少了，但大部分与其基因序列一致。这表明新的驴的基因注释是保守的，可能没有描述完整的驴基因组？？




![此处输入图片的描述][2]
![此处输入图片的描述][3]


**杂合度**
将已有的3种驴和3种斑马重测序数据重新比对到新组装的家驴基因组，发现了这几种马科动物的杂合率趋势与已有研究相似，索马里野驴的杂合性低于家驴，而比对到新组装的驴基因组得到的杂合度低于之前比对到马基因组得到的结果。


![此处输入图片的描述][4]


作者认为，这反映了unidentified/missing CNV sspecific to noncaballine equine species也可以比对到马基因组的相同regions，进而人为的增加了过去杂合率的估算。


  
  
 **群体规模变化**
 
 
 有效种群大小（Ne）的估算取决于潜在的杂合性水平。PSMC模型分析，Onager在~170W年前发生分支并发生群体扩张，接着~70万年出现Somali野驴和家驴的品种分支但群体规模有些许下降。而斑马在~130W年前发生分支。三个斑马物种E. grevyi，E.zebra hartmannae和已灭绝的E.quagga burchellii的PSMC特征从约13,000个个体的祖先种群大小开始在大约〜1.3Ma之前发散。分裂后，与其杂合率一致，Burchell的斑马经历了快速的种群扩张，而格雷维和哈特曼的山斑马的有效种群规模仍然相对较低。
 ![此处输入图片的描述][5]
 
 
 
**共线性分析**


将家驴新组装基因组与马基因组进行共线性分析，确定了多种染色体重排，如易位，倒位，通过排除马基因组组装和驴基因组中scaffold间定序错误后，研究者在马和驴基因组中确定了9个染色体易位和6个染色体倒位，大部分倒位序列与与胚胎发育的代谢和调节以及细胞分裂有关，尤其与中心体功能有关。

![此处输入图片的描述][6]

 
 


  [1]: http://advances.sciencemag.org/content/4/4/eaaq0392.full
  [2]: https://mmbiz.qpic.cn/mmbiz_png/9lFjuMCxfRicG95Fia6fkH44PkMPNZfyxrSB7pzRYW1IS91kMaxXbP12n7UwmJ6QTKdGQdAKJ6k4PMZYSOUC7TJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [3]: https://mmbiz.qpic.cn/mmbiz_png/9lFjuMCxfRicG95Fia6fkH44PkMPNZfyxrowYw1GzrcgSH4ZRsL9oSnOF7uIibiaWEEJehA214CVFeB3rBzkwu0I8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [4]: https://mmbiz.qpic.cn/mmbiz_png/9lFjuMCxfRicG95Fia6fkH44PkMPNZfyxrOyiaUYlfjVMkeCO3fFyRlV9Jkt0ib5hiaDh4o5GMpozdeeQb3JHhlB1OA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [5]: https://mmbiz.qpic.cn/mmbiz_png/9lFjuMCxfRicG95Fia6fkH44PkMPNZfyxrvGxZNm3ibUIuiaI6c912nnrlIpwgGibVrp24hNYXBAf2oZ8v2iacKch8icg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
  [6]: https://mmbiz.qpic.cn/mmbiz_png/9lFjuMCxfRicG95Fia6fkH44PkMPNZfyxrh6ibLCrazlw4FaIyicgZFfBzniaXJ8jVeH1gjT77Vq2eVYQnNK4TeWpbw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1
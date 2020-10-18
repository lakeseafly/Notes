# Week37:甘蔗基因组


标签（空格分隔）： 文献阅读

---

**题目：** Allele-defined genome of the autopolyploid sugarcane Saccharum spontaneum L [Link][1]

**摘要**



现代甘蔗是多倍体种间杂种，结合甘蔗的高糖含量与甘蔗自发性的抗寒性，抗病性和再生性。本文对单倍体S. spontaneum，AP85-441的测序促进了32个假染色体的组装，所述假染色体包含8个同源组，每个4个成员，带有35,525个具有等位基因的基因。 S. spontaneum中基本染色体数目从10减少到8，是由2个祖先染色体的裂变和随后易位到4个染色体引起的。令人惊讶的是，80％与抗病相关的核苷酸结合位点编码基因位于4个重排的染色体中，51％位于重排区域。 64个S. spontaneum基因组的重测序确定了重排区域的平衡选择，保持了它们的多样性。现代甘蔗中的自发性S. spontaneum染色体随机分布在AP85-441基因组中，表明不同S. spontaneum种质中同源物之间的随机重组。等位基因定义的Saccharum基因组为加速甘蔗改良提供了新的认识和资源。



**结果**

**基因组组装**
组装策略overview：

![][2]


 - 35,156 BAC clones，组装软件ALLPATH-LG8 , SPAdes9 和SOAPdenovo，组装结果2.56 Gbp assembly，contig N50 7.4 kb
 - 295 Gbp PacBio RS II三代数据，组装软件CANU，外加BAC数据产生3.13 Gbp 基因组， contig N50 45 kb 。
 - 1 billion of 150 bp PE Hi-C 数据，组装软件ALLHIC，产生32 pseudo-chromosomes， 挂载染色体上基因组序列达 2.9 Gbp，包含97% 的基因。



![][3]
 
 **组装评估**
 
染色单体热图，同一个染色体上的同源染色单体，都具有很好的线性关系。

![][4]
 
与高粱共线性比较，一组4个同源染色体与单个高粱染色体对齐。 S. spontaneum中基本染色体从10减少到8是由染色体裂变引起的，其后是与高粱染色体5和8同源的两个祖先染色体的易位。
 
 ![][5]

 - 遗传图： 89% contigs与Hi-C一致。
 - CEGMA: 219 (88.3 %) CEGs完整。BUSCO:1,397 (97.01 %) 完整。
 - remapping二代数据remma：1,624 144 million (97.01%) 二代数据覆盖 97.3 %基因组。

**基因组注释**
基因注释
 - 两轮的MAKER注释流程，获得 35,525基因，包括 4,289 (12.7%) genes with four alleles, 154 9,792 (27.6%) with three, 14,797 (41.7%) with two, and 6,647 (18.7%) with one.90.0% 注释的基因可以在高粱中找到。
![][6]
 TE注释（can refer to https://www.nature.com/articles/nature15714#methods）
 - RepeatModeler + RepeatMasker
 - unkowb TE: TEclass
 - tandem repeats: TRF package (‘1 1 2 80 5 200 2,000 –d –h’)
 - Telomeres and centromeres :
  more than ten monomers ‘AAACCT’ were identified as telomeres.To identify the centromeric repeats, the largest repeat arrays (period length X copy number) were identified and clustered.


**染色体数目降低与多倍化**
甘蔗由10 变成 8 条原因是经历一些染色体重组。

![][7]
通过比较四个同源染色体A, B, C and D，鉴定了 7.7 million SNPs, 1.03 million short indels and 3,637 SVs。

![][8]

进一步分析研究了等位基因的表达模式，结果发现不同单倍型表达模式相似，并无明显差异；对甘蔗C4光合作用的研究中鉴定了24个基因，7种关键酶参与NADP-MEC4途径；在甘蔗中蔗糖对的积累的研究中发现与高粱相比甘蔗中的液泡膜糖转运蛋白（TSTs，一种蔗糖转运蛋白）发生了基因家族的扩张，所以推测TSTs参与蔗糖在液泡中的积累；S.spontaneum为现代栽培甘蔗提供了抗病性基因，在对其研究中发现，甘蔗中的80％的NBS编码基因位于四个重排染色体（Ss02，Ss05，Ss06和Ss07）上，其中51％位于重排区域。


**群体遗传**

通过对64份甘蔗（Saccharum spontaneum）群体材料遗传多样性研究，发现其具有广泛的自然部分范围，遍及亚洲，印度次大陆，地中海和非洲，且自然种群具有广泛的表型，遗传和倍性水平多样性


![][9]


  [1]: https://www.nature.com/articles/s41588-018-0237-2#data-availability
  [2]: https://5b0988e595225.cdn.sohucs.com/images/20181009/fcd3a89a2a9d45b0b0139373f85059d5.jpeg
  [3]: https://mmbiz.qpic.cn/mmbiz_png/N3X4LBoaQjVom0mlxbHya7d6UJ3GX08ItP811MibGjMHGTGLe9xsdhm3RBGmfia0I0pFgyvDqVXBYkgLw8SppibRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [4]: https://mmbiz.qpic.cn/mmbiz_png/N3X4LBoaQjVom0mlxbHya7d6UJ3GX08ITSIsGGTwhXB8DiaShibkhlmyL8KCp5MctKJ9icpFBdNZPt7lqJ93h6SpA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [5]: https://media.springernature.com/lw900/springer-static/image/art:10.1038/s41588-018-0237-2/MediaObjects/41588_2018_237_Fig1_HTML.png
  [6]: https://mmbiz.qpic.cn/mmbiz_png/N3X4LBoaQjVom0mlxbHya7d6UJ3GX08If6GcO2jEoyQ80Y9qUw4dS4VoIUbHf5WAgpJicHAf5btdAicsD00p5y2g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [7]: https://media.springernature.com/lw900/springer-static/image/art:10.1038/s41588-018-0237-2/MediaObjects/41588_2018_237_Fig2_HTML.png
  [8]: https://media.springernature.com/lw900/springer-static/image/art:10.1038/s41588-018-0237-2/MediaObjects/41588_2018_237_Fig3_HTML.png
  [9]: https://5b0988e595225.cdn.sohucs.com/images/20181009/0003dc84ca4b44f9ae4186a027c3d2e1.jpeg
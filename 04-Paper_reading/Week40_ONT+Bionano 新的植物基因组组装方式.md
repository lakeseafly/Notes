# Week40:ONT+Bionano 新的植物基因组组装方式

标签（空格分隔）： 文献阅读

---

最近发在Nature Plant上的一篇文章巧妙运用ONT结合光学图谱的方法，获得了染色体级别的两种Brassica和一种香蕉的高质量参考基因组，这为植物基因组学研究开辟了一条新的路径。

**Title：** Chromosome-scale assemblies of plant genomes using nanopore long reads and optical maps


**背景**

随着技术的成熟，组装策略也越来越灵活，长读长测序技术对植物基因组连续性的提升十分显著。大部分Illumina组装的基因组连续性欠佳，454测序平台虽然在读长上较其他第二代测序技术有明显的优势，但是和Sanger相比仍然小有距离，和PacBio、ONT相比差距十分明显。


![][1]


该文章主要和大家分享使用ONT辅以Bionano光学图谱技术将两种双子叶和一种单子叶植物组装到contig N50＞5Mb的水平，并组装出包含代表全部染色体或染色体臂的scaffolds。

**结果**

对三个物种测序了38-79×深度的Nanopore长读长reads（相当于4.4-8.2×深度的reads超过50Kb）,组装出的长reads基因组连续性很高：不超过1000条contigs，N50在3.8-7.3Mb之间。加入Bionano光学图谱后，最终组装出的基因组contig N50为5.5-9.5Mb，scaffold N50为15.4-36.8Mb。与已经发布的组装数据进行了比较，发现本研究组装的contig N50是之前基因组的100-450倍。相较于已发布数据组装出的446.8Mb基因组，本研究锚定到了528.8Mb，填补了之前研究没有覆盖到的82Mb的区域。另外在香蕉中，一个三条scaffold跨越了两端端粒重复序列和一段4Mb的高密度着丝粒重复区域。大型contigs对于定位着丝粒的重要性，相较于传统的遗传图谱，光学图谱的信息丰度显然更有意义。

![][2]

接着是做基因组经典的分析:
1. TE annotation 

使用长读长组装结果则检测到了更丰富的长散在（LINE）、（LTR retrotransposon）和DNA转座子家族，总之，使用长读长测序组装鉴定到的TE更加完整。研究者指出，读长是提升转座富集元件区域组装的关键因素，进而决定组装的连续性。

2.比对分析

将重测序的119个白菜型油菜和119个甘蓝型油菜序列分别比对到参考基因组和本研究得到的基因组上，分析发现比对到本研究基因组上的序列比例更高。 

3. 基因功能分析
在基因水平上检测到了高度保守的两种Brassica之间的差异，分析了和春化、花期相关的FLC。


**总结**

结合ONT MinION/PromethION、Bionano光学图谱和Illumina的测序策略能够为获得高质量、低花费的植物组装提供了心得思路。三代测序变得越来越重要。




  [1]: https://mmbiz.qpic.cn/mmbiz_jpg/7771tCIgOPCpSsC6AtXticCpibq92fF4h2bvJvs5PYW8LacFdd9YUhpuHBkFHHQGve0GnDFVmnlCgUSVYAeqftbA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
  [2]: https://mmbiz.qpic.cn/mmbiz_jpg/7771tCIgOPCpSsC6AtXticCpibq92fF4h22iaFB0r3ficdicib30KicnUhJz9FF6n7OR4zpy3BgLS2IodUnOcUVCnUqzw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1
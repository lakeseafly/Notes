# Week13-综述：第三代基因组技术对植物基因组组装的影响


**题目：The impact of third generation genomic technologies on plant genome assembly**

[原文链接][1]

**文章重点信息**

 - 植物基因组比脊椎动物重复性更强，组装更复杂
 - Long-read 组装优于 short-read 组装 
 - Long-range scaffolding可组装至染色体水平
 - 重构建单体型允许拼接杂合体和多倍体基因组

**背景**
植物基因组的组装是一个具有挑战性的问题，并且可能比装配脊椎动物基因组更具挑战性。这个挑战是
由于转座元件的高重复性，极端基因组大小，如22 Gb火炬松和20 Gb挪威云杉基因组。又如，一些植物的多倍体性质，如作物的多倍体性质。最近，long read 还有long-range scaffolding的方法（optical mapping , chromosome conformation capture）变得越来越popular。

**Long-read sequencing technologies**

**PacBio：**

 - with an average size of nearly 20 kb and a maximum length of over 60 kb 
 - sequencing error rates of up to 15%, correction with short sequencing read （错误率蛮高，需要通过short read进行校正）
 - improve short-read assemblies by gap closure or scaffolding
 -  first plant genomes assembled from PacBio：Arabidopsis thaliana and Oropetium thomaeum （never reached by short-read assemblies，less gaps represented as Ns in the sequence ）
 

**Oxford Nanopore：**

 - the longest reads are up to 200 kb
 
**Synthetic Long-Reads (SLR)**

 - de novo assembly requires high amounts of short read coverage
 -  assemblies hardly reached N50 statistics of more than 100 kb

**Long-range scaffolding technologies**

**optical mapping**

 - Bionano: generates fingerprints of DNA fragments of up to multiple hundred kb by imaging the patterns of restriction sites, then assembled into genome-wide (consensus) maps
 - combination of sequencing data and optical maps
 -  optical maps can be used to control for errors in the sequence assembly, the sequence contigs can be used to control for errors in the optical maps.

**Hi-C**

 - Hi-C data are complementary and can help bridging different regions in the genome
 -  improved assembly contiguity similar well as compared to optical mapping data

**10X Genomics**

 - no published records of plant genomes assembled with the 10X Genomics 


**Assembly of heterozygous and polyploid genomes**
  
 - FALCON-Unzip, which uses PacBio sequencing data to phase the variation between individual chromosomes already during the assembly of the reads
 - short and long-read sequencing data combined with optical maps, working well for the assembly of polyploid genomes
 - longer reads or molecules will help improving the reconstruction of individual homeologs


 
  [1]: https://www.sciencedirect.com/science/article/pii/S1369526616301315
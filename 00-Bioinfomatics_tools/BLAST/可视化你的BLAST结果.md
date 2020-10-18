# 可视化你的BLAST结果

> 人类已经使用数据可视化技术很长一段时间了，图像和图表已被证明是一种有效的方法来进行新信息的传达与教学。有研究表明，80％的人还记得他们所看到的，但只有20％的人记得他们阅读的。

我们做本地中运行BLAST后，往往会得到以文字形式的BLAST结果。如果我们需要查看比对的确切结果，这会给我们带来一定的烦恼。今天给大家介绍一个网页based的可视化BLAST结果的小工具：**Kablammo**

### 简介

Kablammo可以让你您从Web浏览器创建BLAST结果，并进行交互式可视化。并且你不需要安装任何软件。简而言之，你只需要找到你最感兴趣的比对结果，然后就可以导出一个已经发布就绪的矢量图像。

听起来是不是很吸引人，下面我将会带领大家学习一下如何使用这个powerful的小工具。

### 实例操作

咱们先点击Kablammo的网站。**http://kablammo.wasmuthlab.org/** 然后我会以该网站提供的例子简单介绍一下如何使用该工具。点开网页中how to work 后， 你会见到下面的图。

![1](C:\Users\胡海飞\Desktop\1.PNG)

### BLAST的一些基本参数

首先，先介绍一下BLAST结果里面一些名称。**Query**就是一段你所感兴趣的DNA或者RNA序列，用以搜索的目标序列。（例子，某些生物的基因序列）。

![2](C:\Users\胡海飞\Desktop\2.PNG)

然后是**Subjects**，每一个query都会有一个或者多个从BLAST数据库所匹配到的subjects。

![3](C:\Users\胡海飞\Desktop\3.PNG)

然后在每一个query-subject的配对中，至少会有一个或者多个的比对。该比对，是经过BLAST的算法后得出的，query至少会有部分序列比对到subjects的序列上。如上图，在query和subjects中，就有两段能够比对上的片段。

![4](C:\Users\胡海飞\Desktop\4.PNG)

### 可视化BLAST的结果

接着就是重头戏，如何进行可视化BLAST的结果。首先，你可以移动鼠标，然后点击到其中一段的比对。这时候你可以看到一系列，比对结果的结果参数，例如E value，Bit score， Querry和subjects的起始于结束的位置。接着，你可以点击view alignment去看具体比对的情况，如下图。

![5](C:\Users\胡海飞\Desktop\5.PNG)

比对上的结果的碱基颜色都一一对应，然后没有比对上，gap的部分就是红色表示。

![6](C:\Users\胡海飞\Desktop\6.PNG)

然后，该小工具还提供了一些小的功能，加入你对query序列中3.5kb的位置特别感兴趣。你还可以通过鼠标的滑轮，放大图中query的位置，然后你可以通过刚刚view alignment的按钮，具体看到你所感兴趣片段的比对情况。

![7](C:\Users\胡海飞\Desktop\7.PNG)

假如，你发现了你所感兴趣的序列的位置，你还可以将该可视化得到的图片保存为SVG或者PNG格式的图片。

### 如何可视化我自己的数据

![8](C:\Users\胡海飞\Desktop\8.PNG)

讲了这么多，重点到了，如何可视化我自己的数据呢？没问题，只要轻松点击右上角load results。然后，load results from your computer。这个上传的BLASTresults需要是XML格式，就是我们常说的format 5。如果你不知道如何获取XML格式的BLAST result，没问题，网站中有一个HOW do I do的连接，手把手教你如何获取该格式的结果。

今天介绍就到这，这款小工具对大家有帮助。
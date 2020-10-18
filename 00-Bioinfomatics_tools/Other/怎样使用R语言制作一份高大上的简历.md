# 怎样使用R语言制作一份高大上的简历

标签（空格分隔）： 生信技巧

---

写简历是一件每个人都要经历的事情，无论你是刚毕业的学生，又或者是想跳槽升级的职场老手。一份精简好看的简历能为你加分不少。说到制作简历，我们最关心的事肯定要数怎样将它弄得简洁明了，格式排版一致。另外，随着你阅历的丰富，怎样快速的在原有的简历上添加上新的工作经验，并且保持格式的一致，是我们写简历时常常有的烦恼。

作为一名生物信息学家，我们当然可以通过编程的方式去解决这个问题。今天就和大家分享一下前几天在推特上看到的一个制作简历的流程，让所有人都能制作出高大上吸引人的简历。


### 模板准备

如果你之前没有用过R markdown，首先要在对应的系统下下载好`pan.doc`这个工具。具体的下载就不展开了，可以自行到该网站上查阅https://pandoc.org/installing.html。

工欲善其事，必先利其器，接着下载好作者提供的模板:

```
### 直接通过链接下载
wget https://github.com/nstrayer/cv/archive/master.zip

### git clone 下载

git clone https://github.com/nstrayer/cv
```

下载好后，让我们看看文件夹里都有什么东西:

![][1]

我们主要使用到的文件有四个，从上往下分别是:

 - logo的图片
 - CV的R markdown文件
 - positions.csv文件，储存我们个人信息的文件，我们通过修改这个文件来制作我们自己的简历
 - Resume的 R markdown文件

这里可能有个别小伙伴会问到**CV和resume有什么不同**？简单来说，Resume才是真正意义上的“简历”，而CV(Curriculum Vitae)其实指的是“履历表”，二者的区别具体为：Resume一般篇幅较短，通常为一页，最多为两页，具有广泛的工作经验的人一般可能写到两页。而CV的篇幅很长，至少要两页，最多的可能会达到十页。

接着需要自己的下载好Rmd中用到的R包，另外要更新最新版的`tidyr`包，因为模板里面用到最新版本`tidyr`包的一个函数。不要问我为什么知道，因为我采坑了。

### 填写个性化的文件

下载好模板后，可以根据你个人的信息开始填写个性化的资料了。先用Excel打开`positions.csv`，效果大概如下，并开始根据每个`section`填上你个人的信息,第二列的`in_resume`可以选择是否将内容放到resume上:
![][2]
 
除了`positions.csv`这个文件之外，在`index.Rmd`或者`resume.Rmd`上也含有一些需要我们自己修改的个人信息，例如你的名字，你联系方式，社交媒体，自我介绍等一系列的内容。
 
 ![][3]
 
所有东西准备好后，点击一下R studio中Rmd旁边的`knitr`，不到十秒一份精致简介的简历就做好了。如果你不使用R studio，也可以使用该命令`rmarkdown::render(‘index.Rmd’)`。最后稍稍给大家截图一下我做的成果：

![][4]

### 后记

介绍到这里就差不多结束了，心动不如行动，大家可以根据上面的步骤制作自己的简历。另外我还非常推荐大家去原作者的博客中看看他原汁原味的文章。在他的博文中，他详细介绍了它是如何一步一步，从**有使用R语言进行简历绘制的想法，到如何糅合不同工具**达成他的目的的过程。比起如何制作简历，明白他的**思维方式**，更是值得我们学习的过程。

原推文博客地址：
https://livefreeordichotomize.com/2019/09/04/building_a_data_driven_cv_with_r/


 


  [1]: http://static.zybuluo.com/lakesea/fosoxrubgws4mrcbzquw9ac5/layout_of_files.png
  [2]: http://static.zybuluo.com/lakesea/o8tjbmr6q9asae01mlfbnxp3/Screen%20Shot%202019-10-16%20at%209.33.04%20am.png
  [3]: http://static.zybuluo.com/lakesea/gzmmgv86zmumlzn1xmesebeu/code_to_change.png
  [4]: http://static.zybuluo.com/lakesea/oif9981rinvagoclwb2ujbod/%E6%89%B9%E6%B3%A8%202019-10-16%20121819.png
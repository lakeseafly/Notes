# R包ggsci：一步完CNS级别的图片配色

标签（空格分隔）： 生信杂谈

---

最近在整理修图，往往遇到的问题就是如果给自己的图片配色。往往我自己认为配色很不错图片，一旦发到老板手上，就会被打回让我换一个新的配色。直到前几天，我搜到一个非常不错的R包“ggsci”，终于完美地解决了我的问题，制作出了一份让老板满意的配图。今天的推文，就和大家简单谈谈这个好用的工具。


## 工具简介

**ggsci**提供了一组适合科学期刊，数据可视化，使用的高质量调色板。另外一个优点就是，**ggsci**包中的调色板可直接嵌套到**ggplot2**中使用。对于所有调色板，相应的function分别命名为：

>   
- scale_color_palname()
- scale_fill_palname()

另外该包的另一个特点就是，它将常用的图片配色,按照期刊杂志喜欢的颜色类型封装了起来，我们可以通过简单地function来对我们的图片进行配色：
![][1]


## 实战演练

### 数据准备

首先，我们先用ggplot2,分别画一个散点图和一个直方图。配色就直接采用，ggplot2 default的颜色。

```
library("ggsci")
library("ggplot2")
library("gridExtra")

data("diamonds")

p1 <- ggplot(
  subset(diamonds, carat >= 2.2),
  aes(x = table, y = price, colour = cut)
) +
  geom_point(alpha = 0.7) +
  geom_smooth(method = "loess", alpha = 0.05, size = 1, span = 1) +
  theme_bw()

p2 <- ggplot(
  subset(diamonds, carat > 2.2 & depth > 55 & depth < 70),
  aes(x = depth, fill = cut)
) +
  geom_histogram(colour = "black", binwidth = 1, position = "dodge") +
  theme_bw()
```
散点图：
![][2]

直方图：
![][3]

总体感觉配色怎么样？是不是感觉说不上很难看，但是总感觉就不符合发表高水平文章的配色（对的，这其实就是各位老板内心的想法）。

### ggsci的使用

那么这时候当然就是轮到我们R包**ggsci**的出场。

首先，我们来试一试Nature的配色：

```
p1 + scale_color_npg()
p2 + scale_fill_npg()
```
散点图：
![][4]
直方图：
![][5]

感觉高端了很多有没有？行不喜欢Nature的配色，咱们试一试Science的：

```
p1 + scale_color_aaas()
p2 + scale_fill_aaas()
```
散点图：
![][6]
直方图：
![][7]

这个配色稍微深一些，但是看起来也不错。

最后让我们再试一试柳叶刀的配色：

```
p1 + scale_color_lancet()
p2 + scale_fill_lancet()
```
散点图：
![][8]
直方图：
![][9]
这个配色给我感觉就是挺fresh的。

## 小结

配色是科研作图的一个大难题，使用**ggsci**能够一定程度上快速解决这个问题。当然如果你的老板或者导师不喜欢这里的配色，你可能还要花更多时间去找到合适的配色。另外，我决定以后我文章初稿里面所有的配图都要使用Science的配色，虽然我的文章难发到Science上，但是至少我的图片配色是Science级别的^_^！


## 参考资料

ggsci的介绍资料：https://nanx.me/ggsci/articles/ggsci.html

  [1]: http://static.zybuluo.com/lakesea/b22cxvm3o7tuad67piinb8e4/image.png
  [2]: http://static.zybuluo.com/lakesea/rbe8lujmkcjtgyq9d5sc4z46/p1.png
  [3]: http://static.zybuluo.com/lakesea/ffdophs042ofja7xgt5zl96r/P2.png
  [4]: http://static.zybuluo.com/lakesea/cgx9gt58pydxoskqwy2hif1k/p1.png
  [5]: http://static.zybuluo.com/lakesea/8ozjqy5itg2orzko769qoboy/P2.png
  [6]: http://static.zybuluo.com/lakesea/d229f5tzgikk07zp732t61ne/p1.png
  [7]: http://static.zybuluo.com/lakesea/ehkevwtaxstud9r0yfoxgzx4/P2.png
  [8]: http://static.zybuluo.com/lakesea/5lygewnvqfhwr3spj1etjy9a/p1.png
  [9]: http://static.zybuluo.com/lakesea/szeth0xsiweffnnr6uj7m0cu/P2.png
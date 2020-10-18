# 不需要懂得编程，但却可以使用ggplot2画出论文级别的图？

标签（空格分隔）： 生信工具

---

你有没有遇到过这样的烦恼，你需要画一些论文级别的图，并且你知道R中的ggplot2是一个很好的选择，可以画出符合你要求的图。但是由于你不熟悉ggplot2的使用，你需要上网倒弄一番，了解与你图相关的代码，有时候花了很长时间确还是没有弄得很满意。嗯嗯，这肯定很熟悉吧。最近有一款新的软件，可以交互式地通过操作界面进行ggplot2画图，有没有很吸引人，那么就让我一起来了解认识一下这款软件**esquisse**。

###软件下载与安装
该软件是基于R的一个插件，可以直接在R的界面中下载：

```
install.packages("esquisse")
```

也可以直接下载developer的版本：

```
remotes::install_github("dreamRs/esquisse")
```

###使用示例

####启动插件

在RStudio中，您可以使用Addins菜单：

![][1]

或者在R中：

```
esquisser()
```

####选择数据
如果在没有默认数据的情况下启动addin，则会出现一个窗口，用于data.frame从Global环境中选择一个窗口（如果当前环境下没有`data.frame`存在，将使用来自`ggplot2`内置的数据集）：

![][2]

选择好数据框后，单击底部按钮以启动绘图部分。

![][3]


####开始画图
画图是插件的主要界面和最有趣实用的部分，在下面的示例中，我们使用`diamonds`来自`ggplot2`内置的数据集进行演示：
![][4]

这个工具会根据你数据框的特点自动帮你选取最合适你这个数据的图，然后将对应变量拖进与画图相关的元素的框框中(X,Y,Group等等)。




![][5]

当然如果你要画的图的类型不是它自动帮你选的，你可以去画图类型按钮那里选择自己需要的图。


![][6]

####控制版面
控制版面的功能也相当强大，可以允许你设置标题，表头，也能让你过滤你想要的数据，并且生成画图的代码，一系列非常实用的功能。下面简单讲解一下每一个控制版面所对应的功能。

**绘图选项**
在这里你可以修改绘图外观和参数，菜单中可用的选项取决于绘图类型：
![][7]



**数据过滤**
用于交互式过滤绘图中使用的数据的小部件：
![][9]

**代码的输出**
在此菜单中，您可以检索用于生成该绘图的代码（包含了用于过滤数据的代码)，并将绘图导出到PNG或PowerPoint。你可以将代码复制到剪贴板，或将其插入当前脚本中。

####插件选项
默认情况下，esquisse将启动到对话框窗口（如果在RStudio中），您可以选择使用浏览器（如果您喜欢）或查看器来实用该插件，可以这样：
```
esquisser(viewer = "browser")
esquisser(viewer = "pane")
```

![][10]

介绍就到这里啦，相信大家也领略到了该工具的强大及其好用，下一步当然是你们自己亲自实践，摸索一下这款工具。


  [7]: https://dreamrs.github.io/esquisse/articles/figures/controls-plot-bar6]: https://dreamrs.github.io/esquisse/articles/figures/create-chart-geom.png


  [1]: https://dreamrs.github.io/esquisse/articles/figures/launch-addin.png
  [2]: https://dreamrs.github.io/esquisse/articles/figures/select-data.png
  [3]: http://static.zybuluo.com/lakesea/wlh4q3ehp3ducgn6q58cca9r/Picture1.png
  [4]: http://static.zybuluo.com/lakesea/pp8vttqrsa12d8ss1ahumoun/Picture1.png
  [5]: https://dreamrs.github.io/esquisse/articles/figures/create-chart.png
  [6]: https://dreamrs.github.io/esquisse/articles/figures/create-chart-geom.png
  [7]: https://dreamrs.github.io/esquisse/articles/figures/controls-plot-scatter.png
  [8]: https://dreamrs.github.io/esquisse/articles/figures/controls-plot-scatter.png
  [9]: https://dreamrs.github.io/esquisse/articles/figures/controls-filter.png
  [10]: https://dreamrs.github.io/esquisse/articles/figures/controls-code.png
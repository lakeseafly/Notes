# 生信小技巧：使用linux中的screen命令去同时管理你多个项目

标签（空格分隔）： 生信小技巧

---

###什么是screen？

screen命令是一个再多个进程之间多路复用一个物理终端的全屏窗口管理器。简单来说，他可以让你再linux终端中创建多个桌面，使你可以快速在不同桌面中切换和工作。

###在什么情况下你需要使用screen？

 1. 你有没有为一些长时间运行的任务而头疼？比如系统备份、大数据在服务器之间上传或者下载等等。通常情况下我们都是为每一个这样的任务开一个远程终端窗口，因为他们执行的时间太长了。必须等待它执行完毕，在此期间可不能关掉窗口或者断开连接，否则这个任务就会被杀掉，一切半途而废了。
 2. 又比如如果你想同时在liux终端进行几个project的数据分析，例如你project1要做一个blast比对找到同源基因，你project2需要做一个RNA-seq的比对，porject3需要做一个基因组的注释。你有没有为又希望能够高效的工作，但同时又对如何合理的安排进行这几个操作感觉到无法应对烦恼不止。

在这些情况下，你都可以使用很溜的screen让你的工作变得更加轻松！另外screen有一个很大的好处就是，他的运行时挂在服务器或者电脑的后台，所以当你在学校办公室工作使用screen运行分析到一半，然后你关闭terminal的窗口后回家后，可以通过screen的功能继续你的工作和分析。

###安装使用
一般的linux系统screen都是自带的不需要安装，如果确实没有的话，可以通过下面的命令安装：

```
apt-get install screen (On Debian based Systems)
```

 当你登录了服务器或者使用你自己的linux的电脑时，直接在命令行那里敲screen，就能进入一个新的窗口。
 
```
hhu@zeus-1 ~ $ screen

```

进入该新的窗口中，然后你可以对应做你想要做的事情，例如创建新的文件啊，比对等等。

另外你也可以通过把你想执行的程序放在screen后面：

```
hhu@zeus-1 ~ $ screen vi test.py
```

这样screen创建一个执行vi test.py的单窗口，当你退出vi时也会同时退出该窗口/会话。

当然当年你在其中一个screen中，你也可以创建一个新的screen，即时执行其他的分析：

```
###创建新的窗口
hhu@zeus-1 ~ $ screen

###在键盘中按Ctrl键+a键，之后再按下c键，screen 在该会话内生成一个新的窗口并切换到该窗口。
```

讲了那么多，要是你想出去散散步，将screen的窗口推出但不中断窗口中正在运行的程序，那你要怎么办呢（这一步叫detach）？不用急**在键盘中按Ctrl键+a键+d键**就能帮到你。


好了，散步散完了，又要再次工作了，那怎么样才能回到刚刚你所创建的窗口中呢，很简单使用-r选项就能够了。

```
screen -r
```

好啦，现在疑问又来了，如果我想执行不同projects中不同的分析，我怎么才能创建一个新的窗口并使用我自定义的名称（直接使用screen，系统会给你一个默认的名字不容易区分）：

```
screen -s annotation
screen -s mapping
```

如果你想查看当前创建了有多少个窗口：

```
screen -list


####屏幕输出如下
There are screens on:
        49658.pts-32.zeus-1     (Detached)
        44531.pts-32.zeus-1     (Attached)
        38992.annotation        (Attached)
        83626.pts-38.zeus-1     (Dead ???)
Remove dead screens with 'screen -wipe'.
4 Sockets in /var/run/uscreens/S-hhu.
```

你可能注意到给screen发送命令使用了特殊的键组合C-a。这是因为我们在键盘上键入的信息是直接发送给当前screen窗口，必须用其他方式向screen窗口管理器发出命令，默认情况下，screen接收以C-a开始的命令。这种命令形式在screen中叫做键绑定（key binding），C-a叫做命令字符（command character）。

可以通过C-a ?来查看所有的键绑定，常用的键绑定有：

```
C-a ?	显示所有键绑定信息
C-a w	显示所有窗口列表
C-a C-a	切换到之前显示的窗口
C-a c	创建一个新的运行shell的窗口并切换到该窗口
C-a n	切换到下一个窗口
C-a p	切换到前一个窗口(与C-a n相对)
C-a 0..9	切换到窗口0..9
C-a a	发送 C-a到当前窗口
C-a d	暂时断开screen会话
C-a k	杀掉当前窗口
C-a [	进入拷贝/回滚模式
```

通过上面的学习，相信你已经对screen有所了解，下面就是通过实操，将这个command用得溜起来,让你数据分析的效率提高！

参考链接：
https://ucdavis-bioinformatics-training.github.io/2018-March-Bioinformatics-Prerequisites/tuesday/screen.pdf


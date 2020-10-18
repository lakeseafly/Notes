# 生信小工具之:并行解压神器pigz

标签（空格分隔）： 生信小工具

---

> 做生物信息的同学会相信你们都会遇到这样的情况:你会遇到成白或者上千的sequencing data,然后你需要解压它/或者将你一系列的结果进行压缩，然后你使用你熟悉的gunzip或者gzip,将你所需要解压或者压缩的文件一个一个压缩，这样会耗费很多的时间，效率并不高。

这次推送给大家带来一个并行解压的神器`pigz`。简单来说，由Mark Adler编写的pigz基本上就是并行gzip，可以有效的提高解压或者压缩的速度。默认情况下，它使用与你的处理器所拥有的相同数量的线程进行压缩。


那问题来了，它有多快？我使用pigz和gzip测试了一些较小文件（~8G未压缩）和一个大（~70G未压缩）文件的压缩和解压缩步骤。然后，这些步骤都是在我的服务器中完成的，具有96个内核和1TB RAM的服务器上完成的(不同配置测出的数据会有所不同)。

###安装pigz

```
# 下载pigz的安装包
wget https://zlib.net/pigz/pigz-2.4.tar.gz

# 解压 pigz
tar -xzvf pigz-2.3.4.tar.gz

# 安装 pigz
cd pigz-2.3.4
make

# 将其路径加到环境变量中( ~/.bashrc) 中，使我们可以全局调用这个工具，然后source刷新一下
# export PATH=$HOME/bin/pigz-2.4:$PATH
source ~/.bashrc
```

###压缩八个比较小的文件
每个文件的大小大概为8.7到8.8G

```
# 使用 gzip
time gzip /RAW/Lappan/shotgun_raw/original_data/808*

real    128m17.322s
user    127m15.664s
sys     0m53.892s

# 使用 pigz
time pigz /RAW/Lappan/shotgun_raw/original_data/808*

real    3m17.983s
user    132m10.900s
sys     1m38.176s
```

这里预防你不清楚`real,user,sys`三者的区别，简单解析一下:

 - **real** :从运行开始到结束的时间。这是所有经过的时间，包括其他进程使用的时间和进程花费的时间（例如，如果它等待I / O完成等）。
 - **user**:是进程所花费的CPU的(但不包括内核)时间。这只是执行过程时使用的实际CPU时间。
 - **sys**是是进程在内核中花费的CPU时间。
简单来说user+sys就是你的进程实际使用了多少总的CPU时间。

这里我们单单比较运行的速度看real就好了，可以看到使用了`pigz`之后压缩的速度比`gzip`快了接近40倍。

###解压8个比较小的文件

每个文件大约2.5G
```
# 使用 gzip
time gunzip /RAW/Lappan/shotgun_raw/original_data/808*

real    8m15.766s
user    6m53.032s
sys     1m22.136s

# 使用 pigz
time unpigz /RAW/Lappan/shotgun_raw/original_data/808*

real    4m26.587s
user    5m27.408s
sys     2m3.336s
```

再次解压的速度使用pigz也有一定幅度的提升。

###压缩一个大的文件
文件大小约为75G
```
# 使用 gzip
time gzip /RAW/Lappan/pigz_test.fastq

real    128m10.759s
user    127m19.204s
sys     0m43.732s

# 使用 pigz
time pigz /RAW/Lappan/pigz_test.fastq

real    3m19.088s
user    131m45.328s
sys     1m42.384s
```

###解压一个大的文件

文件大小大约为22G

```
# 使用 gzip
time gunzip /RAW/Lappan/pigz_test.fastq.gz

real    7m53.338s
user    6m50.192s
sys     1m2.652s

# 使用 pigz
time unpigz /RAW/Lappan/pigz_test.fastq.gz

real    4m23.894s
user    5m25.096s
sys     2m8.820s

```

从上面的结果来看`pgiz`表现是相当不错，当你处理大量的数据时，恰当的使用它能够减少很多你无聊等待解压或者压缩的时间，进而提高你的工作效率。当然这也是其中一种方法。还有很多其它方式能够达到相同的效果，例如使用`parallel`+`gzip`, 又或者`Parafly`+`gzip`等等的方式。如果大家感兴趣，在日后的推文可以专门讨论一下并行运行的技巧工具等这个话题。




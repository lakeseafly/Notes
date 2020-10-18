# 可重复的生信分析之三:使用workflowr进行分析记录，数据共享

标签（空格分隔）： 可重复的生信分析

---

继上两篇推文，`docker`和`conda`的介绍之后，将会来到这个系列中最后一篇推文。本次推文的主要内容是，学习强大的R包`workflowr`，利用其进行可重复的生信分析，并且记录实验内容同时实现数据结果共享。

### Workflowr的介绍

R包`Workflowr`结合了识字编程（knitr和rmarkdown）和版本控制（通过git2r的Git）来生成一个包含时间戳记，版本控制和文档化的结果的网页。任何R用户都可以快速轻松地使用它。其设计的初衷是助研究人员以促进有效的进行项目管理，可重复性的分析，同时进行协作和对结果进行共享。

简单来说`Workflowr`包含了三大特点：

 1. 容易管理
  * 提供了项目管理的模板
  * 使用R markdown 记录结果和编写的代码
  * 使用Git对结果和代表进行版本控制
         
 2. 具有可重复性
  * 示用于创建每个结果的代码版本
  * 每个分析在独立的R session中运行
  * 每个分析所使用的R session的信息会被记录

 3. 容易分享性
  * 对生成的结果使用网页化的形式进行展示
  * 通过GitHub来托管生成的网站
  * 旧版本的结果也会有相对应的链接

### Workflowr的安装

为了让你跟上下列的教程，首先你要将`Rstudio`,`Git`还有`pandoc`这三个基本工具安装好。接着你要开通一个属于你自己的Github账号。

基本准备工作完成后，就可以开始下载R包`Workflowr`了。

```
install.packages("workflowr")
```


### 使用Workflowr开展新的项目


首先，打开你的R studio，然后加载`workflowr`。

```
library("workflowr")
## This is workflowr version 1.6.1
## Run ?workflowr for help getting started
```

如果你以前从未在你电脑上创建过Git的存储库，则需要运行以下命令告诉Git你的用户名和电子邮件。Git使用此信息将你对代码所做的更改进行记录。另外，你只需要在每台电脑上运行一次此命令，随后的所有工作流项目都将使用此信息（你也可以随时通过重新运行该命令来随时对其进行更新）。



```
> wflow_git_config(user.name = "lakeseafly", user.email = "21230309@student.uwa.edu.au")
Current Git user.name:	lakeseafly
Current Git user.email:	21230309@student.uwa.edu.au
Other Git settings:
```

    
现在我们可以开始第一个工作项目了：
```
wflow_start("myproject")
```


这将会创建一个名为的目录myproject/，其中包含了不同的子目录，里面含有对应基本文件，并创建了一个初始的 Git repository。简单查看一下生成的文件：

```
myproject/
├── .gitignore
├── .Rprofile
├── _workflowr.yml
├── analysis/
│   ├── about.Rmd
│   ├── index.Rmd
│   ├── license.Rmd
│   └── _site.yml
├── code/
│   ├── README.md
├── data/
│   └── README.md
├── docs/
├── myproject.Rproj
├── output/
│   └── README.md
└── README.md
```

这时你已经有了一个含完整元素的工作流项目。也就是说，你拥有使用主要工作流程程序命令和发布研究网站所需的所有文件。

必需的两个子目录是`analysis/`和`docs/`。这些目录绝对不能从工作流项目中删除。

 - `analysis/`：此目录包含用于执行项目数据分析的所有R Markdown源文件。它还包含一个特殊的R Markdown文件，index.Rmd该文件不包含任何R代码，但将用于生成index.html，你即将生成的网页主页。此外，该目录还包含重要的配置文件_site.yml，你可以使用该文件来编辑网页的主题，导航栏和其他网站外观。
 - `docs/：`此目录包含生成网页所需的所有HTML文件。HTML文件是根据中的R Markdown文件构建的`analysis/`。此外，R Markdown文件创建的任何图片都保存在这里。这些图的每一个都按照以下模式保存：`docs/figure/<insert Rmd filename>/<insert chunk name>-#.png`

该工作流程特定的配置文件是`_workflowr.yml`。它将对所有R Markdown文件一致地应用到工作流重复性检查。另外最关键的设置是`knit_root_dir`，通过修改其设置，能确定将在其中执行文件的目录是`analysis/`。默认设置是在项目的根目录_workflowr.yml（即"."）中执行代码。要改为从`analysis/`中执行代码，请将设置更改为`knit_root_dir: "analysis"`

在可选的目录是`data/，code/`和`output/`。这些目录是推荐用来更好的管理整个项目的，但是如果你觉得它们没有用，可以将其删除。

 - `data/`：此目录用于存储原始数据文件。
 - `code/`：此目录用于可能不适合以R Markdown格式的代码（例如，用于预处理数据或长时间运行的代码）。
 - `output/`：此目录用于处理的数据文件以及从代码和数据生成的其他输出。

该目录中的该`.Rprofile`文件是常规的R脚本，在打开项目时运行一次。它包含调用`library("workflowr")`，确保每次打开`workflowr`程序项目时自动加载工作流程序。
 
 
### 构建网页
 
你会注意到当前`docs/`的目录当前为空。那是因为我们还没有从这些`analysis/`文件中生成对应的网页。这就是我们下一步要做的事情：

```
wflow_build()
```

```
Current working directory: /Users/ricky/myproject
Building 3 file(s):
Building analysis/about.Rmd
Building analysis/index.Rmd
Building analysis/license.Rmd
Summary from wflow_build

Settings:
 make: TRUE

The following were built externally each in their own fresh R session: 

docs/about.html
docs/index.html
docs/license.html

Log files saved in /private/var/folders/33/nmxv8lqd3zg645dkr9s_5w8c0000gn/T/RtmpYlqh1c/workflowr
```

此命令将在其中构建所有R Markdown文件和`analysis/`相应的HTML文件保存在中`docs/`。最后，它在RStudioViewer或默认的Web浏览器中显示网页。

![][1]

另外可以在不构建任何文件的情况下查看站点，请运行`wflow_view()`，默认情况下会显示该文件`docs/index.html`：

```
wflow_view()
```
接下来可以通过修改`index.Rmd，about.Rmd`和l`icense.Rmd`，更加具体细致地描述你的项目。然后运行`wflow_build()`来重新构建HTML文件，并将其显示在RStudio Viewer或浏览器中。


### 发布网页

`workflow`在已发布和未发布的R Markdown文件之间做出了重要区分。在线的网页上已包含已发布的文件；相反，未发布的R Markdown文件的HTML文件只能在本地计算机上查看。由于该示例的项目才刚刚开始，所以没有发布的文件。要查看项目的状态，可以运行`wflow_status()`：

```
wflow_status()
```
```
Status of 3 Rmd files

Totals:
 3 Unpublished

The following Rmd files require attention:

Unp analysis/about.Rmd
Unp analysis/index.Rmd
Unp analysis/license.Rmd

Key: Unp = Unpublished

The current Git status is: working directory clean

To publish your changes as part of your website, use `wflow_publish()`.
To commit your changes without publishing them yet, use `wflow_git_commit()`.
```

这提醒我们，我们的项目有3个R Markdown文件，并且它们都尚未发布（“ Unp”）的状态。此外，它还指导如何发布它们：第一个参数`wflow_publish()`是要发布的R Markdown文件的字符向量。第二个参数`wflow_git_commit()`是你还没有正式发布之前，你所做出改变的版本控制信息。一般来说，commit的信息越详细越好，以便日后我们可以查询为什么我们要做出这些更改。

这里我们尝试一下运行`wflow_publish()`：

```
wflow_publish(c("analysis/index.Rmd", "analysis/about.Rmd", "analysis/license.Rmd"),"Publish the initial files for myproject")
```

```
Current working directory: /Users/ricky/myproject
Building 3 file(s):
Building analysis/index.Rmd
Building analysis/about.Rmd
Building analysis/license.Rmd
Summary from wflow_publish

**Step 1: Commit analysis files**

No files to commit


**Step 2: Build HTML files**

Summary from wflow_build

Settings:
 clean_fig_files: TRUE

The following were built externally each in their own fresh R session: 

docs/index.html
docs/about.html
docs/license.html

Log files saved in /var/folders/33/nmxv8lqd3zg645dkr9s_5w8c0000gn/T//RtmpYlqh1c/workflowr

**Step 3: Commit HTML files**

Summary from wflow_git_commit

The following was run: 

  $ git add docs/index.html docs/about.html docs/license.html docs/figure/index.Rmd docs/figure/about.Rmd docs/figure/license.Rmd docs/site_libs docs/.nojekyll 
  $ git commit -m "Build site." 

The following file(s) were included in commit 2bf51e9:
docs/about.html
docs/index.html
docs/license.html
docs/site_libs/bootstrap-3.3.5/
docs/site_libs/highlightjs-9.12.0/
docs/site_libs/jquery-1.11.3/
docs/site_libs/navigation-1.1/
```

简单解析一下，这一堆信息。`wflow_publish()` 这一命令主要执行了3个步骤：

 1. 使用自定义提交消息提交3个R Markdown文件
 2. 使用`wflow_build()`生成HTML文件
 3. 提交3个HTML文件以及指定网站样式的文件（例如CSS和JavaScript文件）

执行这3个步骤可确保HTML文件始终与R Markdown文件的最新版本同步。手动执行这些步骤将很繁琐且容易出错，但通过`wflow_publish()`可以轻松保持网站页面同步。

现在，当您运行时wflow_status()，它将报告所有文件都已发布并都是最新的。

```
wflow_status()
## Status of 3 Rmd files
## 
## Totals:
##  3 Published
## 
## The current Git status is: working directory clean
## 
## Rmd files are up-to-date
```

### 部署网站

现在我们已经建立了本地的版本控制网页。下一步是将代码放在GitHub上，以便我们可以让我们的网页在线。

所有必需的设置都可以通过工作流功能执行`wflow_use_github()`。唯一需要的参数是您的GitHub用户名：

```
wflow_use_github("yourname")
```

```
Summary from wflow_use_github():
account: lakeseafly
respository: myproject
* The website directory is already named docs/
* Output directory is already set to docs/
* Set remote "origin" to https://github.com/lakeseafly/myproject.git
* Added GitHub link to navigation bar
* Committed the changes to Git

To proceed, you have two options:

1. Have workflowr attempt to automatically create the repository "myproject" on GitHub. This
requires logging into GitHub and enabling the workflowr-oauth-app access to the account
"lakeseafly".

2. Create the repository "myproject" yourself by going to https://github.com/new and entering
"myproject" for the Repository name. This is the default option.

Enter your choice (1 or 2): 1
You chose option 1: have workflowr attempt to create repo
Requesting authorization for workflowr app to access GitHub account lakeseafly
Waiting for authentication in browser...
Press Esc/Ctrl + C to abort
Authentication complete.
Creating repository myproject
*  Created lakeseafly/myproject
To do: Run wflow_git_push() to push your project to GitHub
```

输入你的用户名后，有两个选项。第一个是询问你是否要授权`workflow`程序在GitHub上自动创建存储库。如果你同意，将会在浏览器中新建一个窗口，对你的用户名和密码进行身份验证，然后将权限授予“ workflowr-oauth-app”以访问你的帐户。如果你选择第二项，手动的，那你就需要手动到github上创建对应的存储库

```
> wflow_git_push() 
Please enter your username (Esc to cancel): lakeseafly
Summary from wflow_git_push

Pushing to the branch "master" of the remote repository "origin" 

Using the HTTPS protocol
The following Git command was run:

  $ git push origin master
```
 
 




  [1]: http://static.zybuluo.com/lakesea/lqr26qqsrdij1kslh3kv2dux/Screen%20Shot%202020-04-07%20at%2011.12.35%20pm.png
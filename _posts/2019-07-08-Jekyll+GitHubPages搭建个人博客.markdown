---
layout: post
title:  "Jekyll+GitHubPages搭建个人博客"
date:   2019-07-08 09:44:00 +0800
category: linux
---
***所有操作默认非root用户，本文中用 `lz` 用户。***

`Jekyll`是一个搭建个人博客的不错的选择，具有以下优点：

- 无需数据库、评论功能或频繁的版本更新—只需关注你的内容。
- 只用 Markdown、Liquid、HTML & CSS 就可以构建可部署的静态网站。
- 原生支持自定义链接、分类、静态页、文章以及自定义布局。
- 与GitHub Pages 天然兼容！
- 学习成本低，基本不需要前端知识，只需要掌握基本markdown语法即可。

接下来，我们从无到有，一步步构建一个部署在`GitHub Pages`上的基于`Jekyll`个人博客。

需要：

- PC一台(推荐Linux)
- Python 2.x (ubuntu自带)
- Git (ubuntu自带)
- GitHub 账号

---

## 本地构建Jekyll博客

首先安装`Jekyll`，由于`Jekyll`是基于`Ruby`的，所以要先安装`Ruby`：

```sh
$ cd ~
$ sudo apt-get install ruby-dev  
# 深坑 一定要装ruby-dev而不是ruby
$ sudo gem install bundler jekyll  
# 否则这里会报Failed to build gem native extension.
Successfully installed jekyll-3.8.6
$ jekyll new myblog  
# myblog可以是任何其他自定义名称
Running bundle install in /home/lz/myblog...
...
New jekyll site installed in /home/lz/myblog.
$ cd myblog  # 进入myblog文件夹
$ bundler exec jekyll serve   
# 如果不指定host则默认搭建在127.0.0.1
# 此时只有本机可以访问该构建在本地的博客
# 若希望局域网（甚至如果有公网IP，外网）也可以访问到该页面
# 则可以跟上参数　-w --host=0.0.0.0
# 该参数适用于aliyun等无图形界面的情况
...
Auto-regeneration: enabled for '/home/lz/myblog'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
# 保留这个终端不要关闭，否则jekyll不工作
```

我们在浏览器访问[`127.0.0.1:4000`](http://127.0.0.1:4000/)，就可以看到我们刚才新建的博客的主页了：

![Your_awesome_title](/PNG/Your_awesome_title.png)

此时只有默认生成的一篇post，名为`Welcome_to_Jekyll`，我们可以点进去看看效果：

![Welcome_to_Jekyll](/PNG/Welcome_to_Jekyll.png)

效果还不错~该有的都有--有标题，有日期，有正文，有单行代码，当然还有技术博客必不可少的代码块(with **代码高亮**)；不该有的广告和杂七杂八的东西都没有 ~~(我真没有diss CSDN blog🐶)~~ 。最重要的是，我们不需要关心这些`html`是如何生成的--只需要专注于`markdown`文稿的撰写即可。“你给他markdwon，他还你html”，就这么简单。

以上所有命令安装了`Ruby`和`Jekyll`并用`jekyll`自带的new命令快速生成了一个`minimum`模板的博客。此时`myblog`目录下的文件结构应该如下：

```sh
├─404.html  # 默认的404页面
├─about.md  # 默认的about页面的markdown文本，会被jekyll解析成html
├─_config.yml # 参数配置文件，可以指定markdown解析引擎、代码高亮等等
├─Gemfile  # 可以指定使用的gems及其版本
├─Gemfile.lock  # 记录已安装的版本，保证跨机器运行时的兼容性
├─index.md  # 默认的主页markdown文本，会被jekyll解析成html
├─_posts  # 文件夹，写好的markdown文件放在这里
│    └─2019-07-08-welcome-to-jekyll.markdown  # markdown文件
├─_site  # 文件夹，写好的markdown文件放在这里
└─index.html  # 运行serve命令时会实时生成的文件夹，不需要git同步
```

最重要的就是`_posts`文件夹，所有写好的`markdown`博文都放在此处，我们`cd`进去这个文件夹看看里面有什么：

```sh
$ cd _posts
$ ls
2019-07-08-welcome-to-jekyll.markdown
```

我们可以从这里看出一点端倪，即标准的jekyll markdown博文的文件名应该是`YEAR-MONTH-DAY-title.MARKUP`格式，`MARKUP`可以是jekyll支持的标记语言，当然我们写markdown格式即可。后缀名是`.md`或者`.markdown`都可以被jekyll正确识别，只要保证文件名是`年份-月份-日期-文章标题.markdown/md`这样既可。

说完了合格的`jekyll markdown`文件名再来看看合格`jekyll markdown` 正文格式：

```sh
$ cat 2019-07-08-welcome-to-jekyll.markdown
---
layout: post
title:  "Welcome to Jekyll!"
date:   2019-07-08 10:02:42 +0800
categories: jekyll update
---
You’ll find this post in your `_posts` directory.
...
```

可以看到正文的头部是有一个特殊的block，都由`---`作为开始和结束的标志。block内有4项内容：

- layout: 可以指定这篇文章的渲染样式，对应`_layout`文件夹下的样式html文件，默认`post`即`_layout/post.html`样式。
- title: 即文章的标题，会显示在主页中和博文页面的顶部
- date: `YYYY-MM-DD HH:MM:SS +/-TTTT`格式，指明年月日时分秒和时区，博文会严格按照此处给出的时间按照从新到旧的顺序显示在主页。
- categories: 分类，此处既可以是categories(支持多分类)也可以是category(单分类)。

在这之后就可以用标准`markdown`语法写博客正文了。

我们将默认的`Welcome to Jekyll!`博文删掉，写一篇自己的博文：

```sh
$ rm 2019-07-08-welcome-to-jekyll.markdown
```

此时在之前的`jekyll serve`窗口中可以看到追加了如下输出：

```text
Regenerating: 1 file(s) changed at 2019-07-08 12:22:33
              _posts/2019-07-08-welcome-to-jekyll.markdown
  Jekyll Feed: Generating feed for posts
              ...done in 0.095358878 seconds.
```

然后新建文件：

```sh
$ touch 2019-07-08-宏颜获水.markdown
```

你可以选用自己熟悉的编辑器，比如`vim`、 `gedit`甚至`VS Code`来编辑该文本文件。

文本示例：

```markdown
---
layout: post
title:  "宏颜获水"
date:   2019-07-08 12:26:25 +0800
category: life
---
##### What's your problem ?
#### What's your problem ?
### What's your problem ?
## What's your problem ?
# What's your problem ?
## What's your problem ?
### What's your problem ?
#### What's your problem ?
##### What's your problem ?
```

编辑完后，保存并退出。

此时在之前的`jekyll serve`窗口中可以看到追加了如下输出：

```text
Regenerating: 1 file(s) changed at 2019-07-08 12:33:45
              _posts/2019-07-08-宏颜获水.markdown
  Jekyll Feed: Generating feed for posts
              ...done in 0.118106864 seconds.
```

可能会有2个和上面十分相似的输出，这是因为`Jekyll`会一直跟踪`_post`文件夹，一旦有更新就会实时渲染新页面，所以这2次输出一次对应`touch`命令创建文件，一次对应编辑后的保存命令。

在浏览器访问[`127.0.0.1:4000`](http://127.0.0.1:4000/)，就可以看到我们刚才新建的宏颜获水了：

![宏颜获水](/PNG/宏颜获水.png)

关于`markdown`语法后续博客会有较为详细的介绍，这里先不展开讲。到这里本地的博客已经搭建完毕，接下来要做的就是将其构建在`GitHub Pages`上，这样可以`DDNS`，而且从任何地方都能访问。

---

## 构建GitHub Pages

首先登陆[GitHub](https://github.com/)，如果没有账号先注册账号，如果有账号点击右上角的`+`，选择`New repository`，在接下来的页面中填写博客项目的`Repository name`，前缀可以自定义，但是后缀必须是`.github.io`，这是GitHub Pages的规定，只有这样GitHub 才会把这个`repo`当作一个GitHub Pages来构建。`Description`和`README`是可选的，可以不填写，直接点击下方的 **Create repository** 按钮，如下图所示：

![Create_a_new_Repository](/PNG/Create_a_new_Repository.png)

这样，一个名为`{GitHub用户名}/{repo名}.github.io`的`repo`就创建成功了。之后页面会自动跳转到刚刚创建的`repo`主页。此时这里是没有任何文件的(如果有那一定是`README`)。我们接下来要做的就是利用`Git`工具将本地的`myblog`文件夹`push`到GitHub的`repo`中。

假定我们已经安装了`Git`工具，并且已经[将本机的公钥添加到了GitHub](https://www.cocobolo.top/linux/2019/04/24/Git-and-GitHub.html)中：

```sh
$ cd ~/myblog
$ git init
# 会在myblog下创建一个repo
Initialized empty Git repository in /home/lz/myblog/.git/

$ git remote add origin https://github.com/{GitHub用户名}/{repo名}.github.io
# 添加远程GitHub repo

$ git add .
# 将myblog下所有文件都添加到本地repo中

$ git status
# 查看当前git状态
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

new file:   .gitignore
new file:   404.html
new file:   Gemfile
new file:   Gemfile.lock
new file:   _config.yml
new file:   "_posts/2019-07-08-\345\256\217\351\242\234\350\216\267\346\260\264.markdown"
new file:   about.md
new file:   index.md

$ git commit -m "add a project to github"
# 如果是第一次运行git commit此时会报错，要求设置邮箱和用户名。按照提示设置后再运行一次commit即可。
[master (root-commit) 87afcfc] add a project to github
 8 files changed, 225 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 404.html
 create mode 100644 Gemfile
 create mode 100644 Gemfile.lock
 create mode 100644 _config.yml
 create mode 100644 "_posts/2019-07-08-\345\256\217\351\242\234\350\216\267\346\260\264.markdown"
 create mode 100644 about.md
 create mode 100644 index.md

$ git push -u origin master
# 这一步需要GitHub访问权限，否则会报错
Warning: Permanently added the RSA host key for IP address '13.229.188.59' to the list of known hosts.
Counting objects: 11, done.
Compressing objects: 100% (10/10), done.
Writing objects: 100% (11/11), 3.44 KiB | 0 bytes/s, done.
Total 11 (delta 0), reused 0 (delta 0)
To git@github.com:Artificial-Ridiculous/myblog.github.io.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

刷新`GitHub`的`repo`页面，可以看到本地的文件和文件夹都被成功上传了：

![new_myblog](/PNG/new_myblog.png)

此时可以浏览器输入[https://{GitHub用户名}.github.io](https://Artificial-Ridiculous.github.io)来访问博客。

如果你对固定的`github.io`域名不满意，下一节将介绍添加自定义域名，将允许通过自定义域名访问你的GitHub Pages博客。

---

## 添加个人域名&开启HTTPS加密

在repo主页上方有一栏选项，最后一项是`setting`，我们点进去往下滑，可以看到一个名为`GitHub Pages`的配置区域。如果我们有自己的域名([阿里云](https://wanwang.aliyun.com/)可以购买，假设是`example.com`)，可以前往DNS设置页面设置一个`CNAME`比如`www`(GitHub强烈推荐使用www子域名)指向`{GitHub用户名}.github.io`：

![CNAME](/PNG/CNAME.png)

然后我们将完整的Custom domain也就是`www.example.com`填入GitHub setting页面的Custom domain输入框，并勾选下方的`Enforce HTTPS`选项开启HTTPS加密：

![www_cocobolo_top](/PNG/www_cocobolo_top.png)

现在就可以通过[https://www.example.com](https://Artificial-Ridiculous.github.io)来访问你的个人博客了，与直接访问[https://{GitHub用户名}.github.io](https://Artificial-Ridiculous.github.io)效果是一样的。

至此，基本的博客框架搭建工作就已经全部结束。如果暂时不想玩出花来(内容价值大于花里胡哨价值)的话，基本上就可以开启`写markdown -> 扔_posts文件夹`的简单写作模式了。

写博客**冰冻三尺非一日之寒**，贵在坚持。加油！与你共勉。


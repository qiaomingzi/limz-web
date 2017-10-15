---
title: Hexo基础使用
date: 2017-09-16 15:42:33
tags:
---

## 什么是 Hexo？

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

## 安装

安装 Hexo 只需几分钟时间，若您在安装过程中遇到问题或无法找到解决方式，请[提交问题](https://github.com/hexojs/hexo/issues)，我会尽力解决您的问题。

### 安装前提

安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：

- [Node.js](http://nodejs.org/)
- [Git](http://git-scm.com/)

如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。

```
$ npm install -g hexo-cli
```

如果您的电脑中尚未安装所需要的程序，请根据以下安装指示完成安装。

> Mac 用户
>
> 您在编译时可能会遇到问题，请先到 App Store 安装 Xcode，Xcode 完成后，启动并进入 **Preferences -> Download -> Command Line Tools -> Install** 安装命令行工具。

### 安装 Git

- Windows：下载并安装 [git](https://git-scm.com/download/win).
- Mac：使用 [Homebrew](http://mxcl.github.com/homebrew/), [MacPorts](http://www.macports.org/) ：`brew install git`;或下载 [安装程序](http://sourceforge.net/projects/git-osx-installer/) 安装。
- Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
- Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`

> Windows 用户
>
> 由于众所周知的原因，从上面的链接下载git for windows最好挂上一个代理，否则下载速度十分缓慢。也可以参考[这个页面](https://github.com/waylau/git-for-win)，收录了存储于百度云的下载地址。

### 安装 Node.js

安装 Node.js 的最佳方式是使用 [nvm](https://github.com/creationix/nvm)。

cURL:

```
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

Wget:

```
$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```

安装完成后，重启终端并执行下列命令即可安装 Node.js。

```
$ nvm install stable
```

或者您也可以下载 [安装程序](http://nodejs.org/) 来安装。

> Windows 用户
>
> 对于windows用户来说，建议使用安装程序进行安装。安装时，请勾选**Add to PATH**选项。
> 另外，您也可以使用**Git Bash**，这是git for windows自带的一组程序，提供了Linux风格的shell，在该环境下，您可以直接用上面提到的命令来安装Node.js。打开它的方法很简单，在任意位置单击右键，选择“Git Bash Here”即可。由于Hexo的很多操作都涉及到命令行，您可以考虑始终使用**Git Bash**来进行操作。

### 安装 Hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```
$ npm install -g hexo-cli
```

变量
全局变量
变量	描述
site	网站变量
page	针对该页面的内容以及 front-matter 所设定的变量。
config	网站配置
theme	主题配置。继承自网站配置。
_ (单下划线)	Lodash 函数库
path	当前页面的路径（不含根路径）
url	当前页面的完整网址
env	环境变量
网站变量
变量	描述
site.posts	所有文章
site.pages	所有分页
site.categories	所有分类
site.tags	所有标签
页面变量
页面（page）

变量	描述
page.title	页面标题
page.date	页面建立日期（Moment.js 对象）
page.updated	页面更新日期（Moment.js 对象）
page.comments	留言是否开启
page.layout	布局名称
page.content	页面的完整内容
page.excerpt	页面摘要
page.more	除了页面摘要的其余内容
page.source	页面原始路径
page.full_source	页面的完整原始路径
page.path	页面网址（不含根路径）。我们通常在主题中使用 url_for(page.path)。
page.permalink	页面的完整网址
page.prev	上一个页面。如果此为第一个页面则为 null。
page.next	下一个页面。如果此为最后一个页面则为 null。
page.raw	文章的原始内容
page.photos	文章的照片（用于相簿）
page.link	文章的外部链接（用于链接文章）
文章 (post): 和 page 布局类似，但是添加了下列变量。

Variable	Description
page.published	如果该文章已发布则为True
page.categories	该文章的所有分类
page.tags	该文章的所有标签
首页（index）

变量	描述
page.per_page	每页显示的文章数量
page.total	总文章数
page.current	目前页数
page.current_url	目前分页的网址
page.posts	本页文章
page.prev	上一页的页数。如果此页是第一页的话则为 0。
page.prev_link	上一页的网址。如果此页是第一页的话则为 ''。
page.next	下一页的页数。如果此页是最后一页的话则为 0。
page.next_link	下一页的网址。如果此页是最后一页的话则为 ''。
page.path	当前页面的路径（不含根目录）。我们通常在主题中使用 url_for(page.path)。
归档 (archive)：与 index 布局相同，但新增以下变量。

变量	描述
page.archive	等于 true
page.year	年份归档 (4位)
page.month	月份归档 (没有前导零的2位数)
分类 (category)：与 index 布局相同，但新增以下变量。

变量	描述
page.category	分类名称
标签 (tag)：与 index 布局相同，但新增以下变量。

变量	描述
page.tag	标签名称

https://wohugb.gitbooks.io     npm使用
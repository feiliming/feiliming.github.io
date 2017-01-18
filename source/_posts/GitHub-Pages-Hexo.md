---
title: 基于GitHub Pages + Hexo搭建静态博客
date: 2016-10-29 16:53:51
tags: 
- Hexo
- GitHub Pages
description: 基于Hexo静态站点生成工具，生成静态博客，使用Markdown语法编写文章，然后发布到GitHub Pages，可以外网独立域名访问，开源、免费、无限制。
---

## 工具/说明

### 1.Windows 7 操作系统
略

### 2.Node.js环境
[Node.js](https://nodejs.org/en/)
大家都知道JavaScript是在客户端浏览器中运行的，而Node.js是让JavaScript脱离浏览器运行在服务器端的平台，使用它可以进行服务器端编程，它遵循CommonJs规范，使用Google Chrome V8的JavaScript引擎，你能想到的它都能干，你想不到的它还能干。

> 感兴趣可以看一下各种Node.js开发指南。

[npm](https://www.npmjs.com/)
npm，node package manager，Node环境下的包管理器，它是一个命令行工具，
npm 使用 package.json 来存储项目的元数据，通过配置文件 package.json 来管理项目的依赖包，类似maven管理项目的pom.xml。

> 如果下载慢指定淘宝镜像，`--registry = http://registry.npm.taobao.org`。

### 3.Git版本控制系统
[Git](https://git-scm.com/)，一个开源免费的分布式版本控制系统。
本文使用的是 [Git for Windows](https://git-for-windows.github.io/) , [TortoiseGit](https://tortoisegit.org/)。 其他更多[GUI Clients](https://git-scm.com/downloads/guis)。

> 注意，别跟GitHub混淆，GitHub是一个项目托管平台，它的版本控制是基于Git实现的。

### 4.GitHub Pages
[GitHub Pages](https://pages.github.com/)，它可以创建用户、组织或者项目的站点，站点是静态的，但它是免费的。一个GitHub账号可以创建一个用户站点，项目站点不受限制，每个项目都可以有一个站点。

> 本文说明怎么创建用户站点，组织站点和用户站点是一样的。

### 5.Hexo
[Hexo](https://hexo.io/)是一个快速、简单且功能强大的静态站点生成框架。Hexo使用Markdown（或其他渲染引擎）解析文章，还以安装很多的第三方插件。

## 方法/步骤

### 1.安装Node.js + npm
下载最新的[Nodejs](https://nodejs.org/en/)，本文使用的是v6.2.0，一路下一步安装，安装Node.js时会自带npm，自带的版本是3.8.9。

如果你安装的Node.js是稳定版v4.4.5，自带的npm版本是v2.15.5，建议升级到最新版本，`npm update npm -g`。

### 2.安装Git
安装 [Git for Windows](https://git-for-windows.github.io/)。
安装 [TortoiseGit](https://tortoisegit.org/)。

### 3.安装Hexo
* 打开Git Bash，全局安装hexo-cli
`npm install -g hexo-cli`
* 切换到工作目录，本文是F盘的gh_blog
`cd f:/gh_blog`
* 执行hexo命令，会在ghb1文件夹下新建所需要的文件，npm install会安装package.json中的包
`hexo init ghb1`
`cd ghb1`
`npm install`
* 此时ghb1目录下文件目录如下：
```
node_modules
scaffolds
source
    _posts
themes
_config.yml
package.json
```
    **node_modules**
    项目已安装的依赖包
    
    **scaffolds**
    放模板的文件夹，文章模板
    
    **source**
    资源文件夹，创建的文章会在这里，解析之后会放到public文件夹
    
    **themes**
    主题文件夹，landscape是默认主题，还可以安装很多主题，hexo会根据主题生成静态页面
    
    **_config.yml**
    博客站点的配置文件，很多参数都在_config.yml配置，包括你安装的插件
    
    **package.json**
    项目依赖信息管理文件，打开它，可以看见已经默认安装了一些插件，比如：[EJS](http://www.embeddedjs.com/)（已经被[DoneJS](https://donejs.com/)替代）、[Stylus](http://stylus-lang.com/)、[Markdown](http://daringfireball.net/projects/markdown/)

### 4.配置_config.yml
配置标题、子标题：
```
title: Feiliming's Website
subtitle: He is not just a coder
```
还有很多可配置的参数，自行参考[docs](https://hexo.io/zh-cn/docs/configuration.html)。

### 5.启动服务
`hexo server`
根据提示访问`http://localhost:4000`
默认有一篇文章，Hello World。

### 6.新建一篇文章
`hexo new post 'GitHub Pages + Hexo'`
post为模板名称，如果不指定则使用_config.yml中配置的default_layout，如果标题有空格，需要使用引号括起来。

> 文件名称和文章title可以不一致。

### 7.编辑文章内容
新建文章后会在/source/_posts/目录下生成对应的*.md文件，.md是使用Markdown语法编写的文件，文本编辑器打开.md文件，使用Markdown语法写一些内容。
再次启动服务`hexo server`，访问主页，发现文章新建成功。

### 8.安装hexo-deployer-git插件
`npm install hexo-deployer-git --save`

### 9.创建一个Repository
GitHub上的个人站点只能有一个，所以Repository的名称必须为`username.github.io`，username为你的GitHub账号名称。

### 10.配置_config.yml的deploy地址
```
deploy:
  type: git
  repo: https://github.com/feiliming/feiliming.github.io.git
  branch: master
```

### 11.生成静态文件
`hexo generate`，会在目录下生成public文件夹，里面是生成的静态文件，即是要部署的文件。

### 12.部署到GitHub
`hexo deploy`，提交到GitHub，执行命令会弹出输入用户名和密码。

### 13.访问GitHub站点
`https://username.github.io/`

### 14.切换主题
打开Git bash，切换到站点目录，然后从GitHub克隆主题，以[hexo-theme-next](http://theme-next.iissnan.com/)主题为例：
`git clone https://github.com/iissnan/hexo-theme-next themes/next`
然后修改站点的_config.yml的主题配置：
`theme : next`
启动服务，发现主题已经切换。

进入theme/next主题目录，会发现主题的配置文件_config.yml，可以修改主题的一些配置。修改Scheme为：
`scheme: Pisces`

### 15.集成文章评论
hexo默认使用disqus评论，改成 [多说](http://duoshuo.com/) 评论：
在多说创建站点，然后将多说域名的前一段写入站点的_config.yml配置文件中，
`duoshuo_shortname: feilm`
其中 `feilm` 是多说域 `http://feilm.duoshuo.com/` 名中的。

> 集成评论后，每个文章页面都会有评论组件，如果不需要则在页面的Front-matter中关闭，`comments: false`。

### 16.新建标签页
`hexo new page "tags"`，发现source目录下新建了/tags/index.md目录文件，编辑index.md的[Front-matter](https://hexo.io/zh-cn/docs/front-matter.html)部分：
```
---
title: tags
date: 2016-11-01 14:12:11
type: "tags"
comments: false
---
```
type指定文件类型，comments是否开启本页的评论。

> Front-matter 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量。

编辑主题目录下的/themes/next/_config.yml配置文件，修改menu的配置项：
```
menu:
  home: /
  categories: /categories
  tags: /tags
  archives: /archives
  about: /about
  #schedule: /schedule
  #commonweal: /404.html
```

> 注意：
1. menu的配置项不会直接显示到站点的菜单上，它会对应某个language.yml文件。语言是通过站点根目录的_config.yml的language指定的：
`language: zh-Hans`
语言对应文件在主题目录中：
`themes/next/languages/zh-Hans.yml`。
2. 编写文章时多个tags、categories写法：
```
tags: 
- Hexo
- GitHub Pages
```


### 17.新建分类页
`hexo new page "categories"`，发现source目录下新建了/categories/index.md目录文件，编辑index.md的Front-matter部分：
```
---
title: categories
date: 2016-11-01 14:12:11
type: "categories"
comments: false
---
```
type为categories指定文件类型，comments为false关闭评论。

> about页同理。

### 18.文章截断
方法1是在文章里，使用` <!--more--> ` 截断；
方法2是在Front-matter中指定description内容，文章列表自动显示description的内容。

> 如果都设置，会优先使用description。

### 19.设置头像
编辑站点的  _config.yml ，新增字段  avatar ， 值设置成头像的链接地址。
其中，头像的链接地址可以是：
1.完整的互联网 URL，例如： https://avatars1.githubusercontent.com/u/32269?v=3&s=460  
2.站点内的地址，例如：
* /uploads/avatar.jpg  需要将你的头像图片放置在  站点的 source/uploads/ （可能需要新建 uploads 目录）
* /images/avatar.jpg  需要将你的头像图片放置在  主题的 source/images/  目录下。

### 20.绑定自己的域名
首先去阿里云买一个自己喜欢的域名，阿里已经把万网收购了
然后添加域名解析设置
```
记录类型    主机记录     解析线路    记录值
A           @            默认        ping yourusername.github.io后的ip
CNAME       www          默认        yourusername.github.io 
```
然后在source目录创建CNAME文件，内容为
```
www.feiliming.com
```

### 21.上传到GitHub后vendors下文件404
Next主题上传到GitHub后vendors下文件404，其他主题无问题
Next主题作者说是GitHub pages屏蔽了vendors目录，解决办法是：
方法1是在repo根目录下创建.nojekyll文件，内容是空的即可
方法2是将主题目录下vendors文件改名，然后将主题配置_configl.yml中的vendors-》_internal改成对应文件名

### 22.添加访问统计
[hexo第三方集成服务](http://theme-next.iissnan.com/third-party-services.html)
本文使用[不蒜子](http://busuanzi.ibruce.info/)
修改主题目录下配置文件_config.yml
```
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"> 本站总访客数</i>
  site_uv_footer: 人
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"> 本站总访问量</i>
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: 本文总阅读量
  page_pv_footer: 次
```

### 23.添加分享
[hexo第三方集成服务](http://theme-next.iissnan.com/third-party-services.html)
本文使用[jiashis](http://www.jiathis.com/)
修改站点目录下配置文件_config.yml
```
# JiaThis 分享服务
jiathis: true
```
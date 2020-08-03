---
title: Hexo-Next实用主题优化
date: 2020-07-23 22:47:32
tags:
  - 博客
categories:
  - 博客
keywords: Hexo, Next, 主题优化
cover: /images/封面图/bilibili.jpg
top_img: /images/封面图/bilibili.jpg
description: 本文介绍Hexo-Next主题的常用主题优化和进阶优化
sticky: 0
---


#  **简介**

本文介绍Hexo博客的Next主题下的一些基础设置和实用优化配置，这些设置也在本文的博客中有所体现。



#  **基础设置**

本节内容介绍hexo博客的一些基础配置，主要是修改一些配置文件中的内容。

##  **博客名称**

修改<span id="inline-blue">站点配置文件</span>：
```
# Site
title: 田宇的个人博客      # 网站标题
subtitle: '学习笔记'        # 网站副标题
description: ''  # 网站描述 主要用于SEO,该部分内容还会出现在侧边栏作者下面
keywords:
author: yu.tian    # 作者名字 用于主题显示文章的作者
language: zh-Hans   # 网站使用的语言
timezone: 'UTC'    # 网站时区 Hexo 默认使用您电脑的时区
```

##  **博客路径**

修改站点配置文件：
```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
## 如果您的网站存放在子目录中，例如 http://yoursite.com/blog，则请将您的 url 设为 http://yoursite.com/blog 并把 root 设为 /blog/
url: http://tianyu-code.top    # 网址
root: /     # 网站根目录
permalink: :title/    # 生成的博客目录包含年月日，不利于搜索，简化
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks

```

##  **每页文章数量**

修改站点配置文件：
```
index_generator:
  path: ''
  per_page: 10      # 每页显示的文章量
  order_by: -date   # 排序：根据时间
```

```
# Pagination
## Set per_page to 0 to disable pagination
per_page: 20    # 每页显示的文章量 (0 = 关闭分页功能)
pagination_dir: page    # 分页目录
```

##  **修改网站主题**

修改站点配置文件：
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next     # 当前主题名称，值为false时禁用主题
```

##  **设置博客主页菜单**

博客主页的各种按钮需要配置才能正确跳转，例如：
<img src=/images/HEXO博客next主题优化/主页按钮.png >


1. 修改主题配置文件，其中`||`符号前是跳转路径，后面是按钮图标
    ```
    # When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash from link value (/archives -> archives).
    # Usage: `Key: /link/ || icon`
    # Key is the name of menu item. If translate for this menu will find in languages - this translate will be loaded; if not - Key name will be used. Key is case-senstive.
    # Value before `||` delimeter is the target link.
    # Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, question icon will be loaded.
    menu:
      home: /|| home
      #about: /about/ || user
      tags: /tags/|| tags
      categories: /categories/|| th
      #archives: /archives/ || archive
      #schedule: /schedule/ || calendar
      #sitemap: /sitemap.xml || sitemap
      #commonweal: /404/ || heartbeat

    # Enable/Disable menu icons.
    menu_icons:
      enable: true
    ```

    > 我在配置时必须把跳转路径后面的空格去掉，不然在跳转的时候空格也会在路径中导致失败

2. 新增分类/标签等目录

    在完成上述配置后，我们指定了按钮的跳转路径，但是这个目录还未存在，需要手动创建,在博客目录输入`hexo new page categories`，即可创建分类目录，会在`source`下面生成`categories`目录，里面的`index.md`即为`categories`界面的显示文件，`index.md`文件修改如下：

    <img src=/images/HEXO博客next主题优化/主页按钮目录配置.png width="40%">

3. 写博客时添加分类

    接下来我们在添加新博客时，注意添加分类，下面给出一个多级分类的例子
      <img src=/images/HEXO博客next主题优化/主页按钮分类测试.png width="70%">

    这样我们点击分类就会自动出来刚刚的那篇文章了
    <img src=/images/HEXO博客next主题优化/主页按钮分类效果.png width="70%">


> 其余页面同理，根据需要设置


##  **设置主题风格**

next主题默认提供了四种风格可选，本博客是Gemini风格，修改主题配置文件：

<img src=/images/HEXO博客next主题优化/主题风格.png width="40%">


##  **侧边栏社交按钮**

侧边栏可添加社交按钮，修改主题配置文件，其中`||`前面是链接，后面是图标
```
# Social Links.
# Usage: `Key: permalink || icon`
# Key is the link label showing to end users.
# Value before `||` delimeter is the target permalink.
# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, globe icon will be loaded.
social:
  GitHub: https://github.com/tianyu-code || github
  E-Mail: mailto:1406985325@qq.com || envelope
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
  #FB Page: https://www.facebook.com/yourname || facebook
  #VK Group: https://vk.com/yourname || vk
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  #Instagram: https://instagram.com/yourname || instagram
  #Skype: skype:yourname?call|chat || skype

social_icons:
  enable: true
  icons_only: false
  transition: false

# Blog rolls
links_icon: link
links_title: Links
links_layout: block
#links_layout: inline
```

效果如下：
<img src=/images/HEXO博客next主题优化/侧边栏社交按钮示例.png width="20%">

##  **侧边栏头像**

将头像的图片放在hexo/source/images下即可（若没有该目录则新建即可）,修改主题配置文件:
```
# Sidebar Avatar 设置头像
# in theme directory(source/images): /images/avatar.gif
# in site  directory(source/uploads): /uploads/avatar.gif
avatar: /images/user.jpg
```


##  **侧边栏其他设置**

### **侧边栏出现时机和位置**
### **回到顶部按钮**

修改主题配置文件:
```
sidebar:
  # Sidebar Position, available value: left | right (only for Pisces | Gemini).
  position: left
  #position: right

  # Sidebar Display, available value (only for Muse | Mist):
  #  - post    expand on posts automatically. Default.
  #  - always  expand for all pages automatically
  #  - hide    expand only when click on the sidebar toggle icon.
  #  - remove  Totally remove sidebar including sidebar toggle.
  display: post
  #display: always
  #display: hide
  #display: remove

  # Sidebar offset from top menubar in pixels (only for Pisces | Gemini). 两个栏的间隔像素点
  offset: 12

  # Back to top in sidebar (only for Pisces | Gemini).   回到顶部按钮是否开启
  b2t: true

  # Scroll percent label in b2t button. 回到顶部按钮是否显示百分比
  scrollpercent: true

  # Enable sidebar on narrow view (only for Muse | Mist).
  onmobile: false
```


##  **主页文章显示设置**

### **阅读更多按钮**
### **记录上次阅读位置**
### **自动滚动到<!--more-->下面**
### **文章标题下面一些元素显示，包括更新时间，创建时间等**

上述设置均修改主题配置文件:

```
# ---------------------------------------------------------------
# Post Settings
# ---------------------------------------------------------------

# Automatically scroll page to section which is under <!-- more --> mark.
# 自动滚动到<!--more-->下面
scroll_to_more: false

# Automatically saving scroll position on each post/page in cookies.
# 使用cookies记录上次阅读位置
save_scroll: false

# Automatically excerpt description in homepage as preamble text.
excerpt_description: true

# Automatically Excerpt. Not recommend.
# Please use <!-- more --> in the post to control excerpt accurately.
# 打开阅读更多按钮
auto_excerpt:
  enable: true
  length: 150

# Post meta display settings
# 文章标题下面的一些元素，可选显示
post_meta:
  item_text: true
  created_at: true
  updated_at: true
  categories: true
```

效果如下:

<img src=/images/HEXO博客next主题优化/文章显示设置示例.png width="70%">

##  **文章版权声明**

修改主题配置文件:
```
# Declare license on posts
post_copyright:
  enable: true
  license: CC BY-NC-SA 3.0
  license_url: https://creativecommons.org/licenses/by-nc-sa/3.0/

```

效果如下，注意它生成的链接是根据站点配置中的URL生成的,效果图：

<img src=/images/HEXO博客next主题优化/文章版权说明示例.png width="70%">

##  **代码块高亮风格设置**

next主题自带tomorrow代码高亮，其中包含五种风格，修改站点配置文件:
```
highlight:      # 代码块的设置
  enable: true
  line_number: true
  auto_detect: true    # 自动检测代码类型
  tab_replace: ''
  wrap: true
  hljs: false
```

修改主题配置文件:
```
# Code Highlight theme
# Available value:
#    normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: night eighties

```


注意在写博客时，需要使用如下格式才能触发代码高亮

\```C

code...

\```

##  **页脚设置**
博客最下方的页脚有些杂乱，比如由Hexo强力驱动、hexo版本等，为了简洁我们可以去掉，修改主题配置文件：
```
footer:
  # Specify the date when the site was setup.
  # If not defined, current year will be used.
  since: 2020

  # Icon between year and copyright info.
  icon: user

  # If not defined, will be used `author` from Hexo main config.
  copyright:
  # -------------------------------------------------------------
  # Hexo link (Powered by Hexo).
  powered: false

  theme:
    # Theme & scheme info link (Theme - NexT.scheme).
    enable: false
    # Version info of NexT after scheme info (vX.X.X).
    version: false
  # -------------------------------------------------------------
  # Any custom text can be defined here.
  #custom_text: Hosted by <a target="_blank" href="https://pages.github.com">GitHub Pages</a>
  custom_text: 田宇的个人博客
```
效果图：

<img src=/images/HEXO博客next主题优化/页脚设置示例.png width="30%">


##  **添加评论系统**

综合考虑下来，决定使用valine系统，它的优点如下：
（1）有国内网站，加载速度快，Leancloud管理评论也方便
（2）无需再使用第三方登录，可游客直接评论，直接设置自己的昵称即可

1. 进入[Leanclound](https://www.leancloud.cn/),注册登录后创建应用，如下所示：

    <img src=/images/HEXO博客next主题优化/添加评论之创建应用.png width="60%">

    进入刚才创建好的应用，在`“存储”`中选择`“创建Class”`，设定Class名称为`Comment`，设置ACL权限为创建者可读可写，如下图

    <img src=/images/HEXO博客next主题优化/添加评论之创建class.png width="60%">

    然后进入“设置”中的“安全中心”,添加Web安全域名，防止其他用户盗用

    <img src=/images/HEXO博客next主题优化/添加评论之安全设置.png width="70%">

    再进入`“设置”`中的`“应用Keys”`，记录`App ID和AppKey`的值

2. 修改主题配置，将App ID和AppKey的值填入

    <img src=/images/HEXO博客next主题优化/添加评论之修改配置文件.png width="70%">

    `placeholder`是在评论框中显示的提示标语，默认`Just go go`可自行修改
    `guest_info`是评论上方显示的头部
    添加成功后文章标题下访还会有评论数的统计

    效果如下：
    <img src=/images/HEXO博客next主题优化/添加评论效果.png width="70%">

3. 评论管理

    管理评论在leanClound中即可
    <img src=/images/HEXO博客next主题优化/添加评论之评论管理.png>


##  **本地搜索功能**

在博客根目录安装
`npm install hexo-generator-searchdb --save`

站点配置文件添加如下:
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

主题配置文件修改如下：

```
# Local search
# Dependencies: https://github.com/flashlab/hexo-generator-search
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1

```

效果图：

<img src=/images/HEXO博客next主题优化/本地搜索效果图.png width="70%">


##  取消主题默认动画

next主题的内容和菜单都有相关的动画，影响加载速度，可以在主题配置文件中取消，将motion的enable改为false即可
```
#! ---------------------------------------------------------------
#! DO NOT EDIT THE FOLLOWING SETTINGS
#! UNLESS YOU KNOW WHAT YOU ARE DOING
#! ---------------------------------------------------------------

# Use velocity to animate everything.
motion:
  enable: false
  async: false
  transition:

```

---

#  **主要优化选项**

该部分设置涉及了一些主题的css设置，影响了博客内容的样式等。

##  **添加文章结束标识**

在目录 `themes/next/layout/_macro/` 下添加 `passage-end-tag.swig`文件，内容如下：

```
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ff4d4d;font-size:16px;">--------------------- 本文结束,感谢您的阅读---------------------</div>
    {% endif %}
</div>
```

打开 `themes/next/layout/_macro/post.swig` 文件，新增内容如下:

```
<div>
    {% if not is_index %}
    {% include 'passage-end-tag.swig' %}
    {% endif %}
 </div>
```

打开主题配置文件 ，添加代码如下：

```
# 文章末尾添加“本文结束”标记
passage_end_tag:
enabled: true
```

##  **将乱码的上一页/下一页修正**

next主题存在一个bug，就是当首页文章过多而分页时，上一页和下一页会出现乱码，如图所示：


<img src=/images/HEXO博客next主题优化/翻页bug图.png width="70%">

修改`themes/next/layout/_partials/pagination.swig`文件：
```
{% if page.prev or page.next %}
  <nav class="pagination">
    {{
      paginator({
        prev_text: '上一页',
        next_text: '下一页',
        mid_size: 1
      })
    }}
  </nav>
{% endif %}

```

修复后效果图：
<img src=/images/HEXO博客next主题优化/翻页bug修复后.png width="50%">


##  **修改文章链接样式**

打开`themes/next/source/css/_common/components/post/post.styl`，在开头添加

```
.post-body p a {
  color: #0593d3;
  border-bottom: none;
  border-bottom: 1px solid #0593d3;
  &:hover {
    color: #fc6423;
    border-bottom: none;
    border-bottom: 1px solid #fc6423;
  }
}
```

效果测试：[个人博客](tianyu-code.top)


##  **常用CSS样式修改**
###  **更改背景图片**
###  **简单代码块样式**
###  **归档和分类页面的时间轴样式**
###  **修改archives页面移动端菜单下拉的bug**
###  **博客中引用的样式**
###  **选中文字部分的样式**

以上所有的设置均在统一个文件中修改：`hexo/themes/next/source/css/_custom/custom.styl`，该文件一开始默认是空，可新增我们想要的css样式（估计大部分的css修改都在此文件夹）.
下面附上我的文件修改，各种颜色，透明度等细节修改见注释：

```
// Custom styles.

// 页面整体设置

body {
	//background-image:url(https://source.unsplash.com/random/daily);
	background-image:url(/images/background.jpg); //这一行的括号里填背景图片的路径，将图片重命名为background.jpg放在\themes\next\source\images下
	background-repeat: no-repeat;
	background-attachment:fixed;
	background-position:50% 50%;
	background-size:100% 100%;//默认将图片和当前窗口大小100%填充，强迫症福利
}


// ``简单代码块的自定义样式

code {
    color: #000;
    background: rgba(255,0,0,.3);
    margin: 1px 3px;
    padding: 1px 3px;
}

// 归档和分类页面的时间轴样式

.posts-collapse {
    margin: 50px 0;
}
@media (max-width: 767px) {
    .posts-collapse {
        margin: 50px 20px;
    }
}

.posts-collapse::after {
    margin-left: -2px;
    background: rgba(255, 118, 0, 0.75);
}

.posts-collapse .post-title a {
    color: #0095ff;
}

.posts-collapse .post-title a:hover {
    color: #ff7600;
}


// 修改archives页面移动端菜单下拉的bug

.posts-collapse {
    z-index: initial;
}



// 博客中引用的样式
blockquote {
    padding-top: 3px;//引用框和字体上部的填充空白
    padding-right: 20px;//右边的空白
    padding-bottom: 0px;//底部的空白
    padding-left: 20px;//左部的空白
    border-left: 5px solid rgba(255, 118, 0, 0.75);//竖条的属性（宽度和颜色rgba，a为透明度）
}

// 选中文字部分的样式
::selection {
    background: rgb(46, 124, 160);
    color: rgb(255, 255, 255);
}
*::-moz-selection {
    background: rgb(46, 124, 160);
    color: rgb(255, 255, 255);
}
```

##  **Gemini主题透明化**

效果图：

<img src=/images/HEXO博客next主题优化/界面布局修改前.png width="70%">

> 个人倾向于设定为0.9，这样不影响内容阅读，且能模糊的看见背景缓解视觉疲劳

1. 内容板块透明   
    `themes/next/source/css/_schemes/Pisces/_layout.styl` 文件 `.content-wrap` 标签下修改：
    `background: rgba(255,255,255,0.9); //0.9是透明度`

    `themes/next/source/css/_schemes/Gemini/index.styl`文件 `.post-block` 标签下修改：
    `background: rgba(255,255,255,0.9); //0.9是透明度`

2. 分页(主页最下面的那一小块)   
    `themes/next/source/css/_schemes/Gemini/index.styl` 文件 `.pagination` 标签下修改：
    `background: rgba(255,255,255,0.9); //0.9是透明度`

3. 菜单栏背景   
    `themes/next/source/css/_schemes/Pisces/_layout.styl` 文件 `.header-inner` 标签下修改：
    `background: rgba(255,255,255,0.9); //0.9是透明度`

    `themes/next/source/css/_schemes/Pisces/_sidebar.styl` 文件 `.sidebar` 标签下增加一行:
    `opacity: 0.9; // 0.9透明度自己选择`

4. 站点概况背景   
    `themes/next/source/css/_schemes/Pisces/_sidebar.styl`文件 `.sidebar-inner` 标签下修改：
    `background: rgba(255,255,255,0.9); //0.9是透明度`

    `themes/next/source/css/_schemes/Pisces/_layout.styl` 文件 `.sidebar` 标签下修改为：
    `background: rgba(255,255,255,0.9); //0.9是透明度`

5. 评论区背景   
    `themes/next/source/css/_schemes/Gemini/index.styl`文件 `.comments` 标签下修改为：
    `background: rgba(255,255,255,0.9); //0.9是透明度`


##  **文章目录自动全展开**

使用Next主题写文章时, 只有当你浏览到相应的目录级时才会展开。想通过目录快速定位某段文章的时候就很不方便，修改如下:
打开文件`hexo/themes/next/source/css/_common/components/sidebar/sidebar-toc.styl`

```
找到如下的代码：
.post-toc .nav .nav-child { display: none; }
修改为：
.post-toc .nav .nav-child { display: block; }
```

##  **修改Gemini风格的博客界面布局，即修改界面宽度和留白**

打开文件`hexo/themes/next/source/css/_variables/Gemini.styl`

```
// Settings for some of the most global styles.
// --------------------------------------------------
$body-bg-color                    = #eee
$main-desktop                     = 90%      //代表整体占用浏览器界面的百分比，即控制留白大小
$sidebar-desktop                  = 240px   //侧边栏出现的位置
$content-desktop                  = calc(100% - 250px)    //内容占用的大小
```
修改前后对比图：

<img src=/images/HEXO博客next主题优化/界面布局修改前.png width="70%">

<img src=/images/HEXO博客next主题优化/界面布局修改后.png width="70%">


##  **修改字体**

这里只是简单的改一下字体大小，格式什么的就不动了，文件：`themes/next/source/css/_variables/base.styl`
```
// Font size
$font-size-base           = 15px    //字体大小
$font-size-base           = unit(hexo-config('font.global.size'), px) if hexo-config('font.global.size') is a 'unit'
$font-size-small          = $font-size-base - 2px
$font-size-smaller        = $font-size-base - 4px
$font-size-large          = $font-size-base + 2px
$font-size-larger         = $font-size-base + 4px
```

##  **使用不蒜子统计站点访问量**

编辑主题配置文件
```
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i>本站总访客数
  site_uv_footer: 人次
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i>本站总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: 本文总阅读量 # <i class="fa fa-file-o"></i>
  page_pv_footer: 次
```

由于不蒜子的域名有更改，所以我们也需要在next主题中修改：
`/theme/next/layout/_third-party/analytics/busuanzi-counter.swig`文件:

src替换为"https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"。

---


#  **可选优化选项**

该部分设置锦上添花，让你的博客变得更美观和有意思。


##  **图片懒加载**

当博客图片较多时，使用懒加载可大大加快加载速度，而且它只会加载当前界面的图片，后续图片当滚动到时才会进行加载。

1. 在博客根目录执行命令安装插件：`npm install hexo-lazyload-image --save`.
2. 在站点配置文件添加配置

    ```
    #懒加载设置
    lazyload:
      enable: true
      onlypost: false
      loadingImg:    # /images/loading.png
    ```
    `onlypost`写成false代表所有的图片都启用懒加载，否则就是仅文章中的图片
    `loadingImg`是未加载时的默认图片，放到主题的目录`next/source/images`目录下，若不填写则为默认

##  **添加界面宠物**

该配置可在博客页面添加一只宠物，例如：
<img src=/images/HEXO博客next主题优化/宠物.png width="40%">

1. 在博客根目录安装live2d插件：`npm install --save hexo-helper-live2d`
2. 在博客根目录安装指定的宠物模型：`npm install live2d-widget-model-shizuku`

    > 对应模型效果可参考[宠物模型效果](https://huaji8.top/post/live2d-plugin-2.0/)
3. 在站点配置文件添加配置：

    ```
    # live2D动画
    live2d:
      enable: true
      scriptFrom: local
      pluginRootPath: live2dw/
      pluginJsPath: lib/
      pluginModelPath: assets/
      tagMode: false
      debug: false
      model:
        use: live2d-widget-model-shizuku # 指明使用哪个动画
      display:
        position: left
        width: 150    #宽度
        height: 240   #高度
        hOffset: 100  # 调节水平位置
        vOffset: 0    # 调节垂直位置
      mobile:
        show: false   #移动端是否显示
      react:
        opacity: 0.8   #不透明度
    ```

上述配置呈现的效果即为当前博客中的宠物。[个人博客链接](tianyu-code.top)


##  **添加实时沟通**

尽管添加了评论系统，但是沟通效果还是差强人意，这里我们使用daovoice，能在博客界面添加实时沟通，将消息发送到作者的微信

1. 注册登录
    首先进入[daovoice](http://dashboard.daovoice.io)注册，然后通过邀请码[c54c97de](http://dashboard.daovoice.io/get-started?invite_code=c54c97de)激活。之后找到APPID

    <img src=/images/HEXO博客next主题优化/实时沟通注册.png>


2. 修改配置文件

    打开themes/next/layout/_partials/head.swig文件，添加如下代码：
    ```
    {% if theme.daovoice %}
      <script>
      (function(i,s,o,g,r,a,m){i["DaoVoiceObject"]=r;i[r]=i[r]||function(){(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;a.charset="utf-8";m.parentNode.insertBefore(a,m)})(window,document,"script",('https:' == document.location.protocol ? 'https:' : 'http:') + "//widget.daovoice.io/widget/0f81ff2f.js","daovoice")
      daovoice('init', {
          app_id: "{{theme.daovoice_app_id}}"
        });
      daovoice('update');
      </script>
    {% endif %}
    ```
    <img src=/images/HEXO博客next主题优化/实时评论修改配置文件.png>

    然后在主题配置文件最后添加配置
    ```
    # Online contact 
    daovoice: true
    daovoice_app_id: 这里填你的刚才获得的 app_id
    ```
    之后就可以看到图标了，这个可能加载较慢

    

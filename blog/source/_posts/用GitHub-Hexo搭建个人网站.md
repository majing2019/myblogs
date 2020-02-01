---
title: 用GitHub+Hexo搭建个人网站  #文章页面上的显示名称，可以任意修改
date: 2020-02-01 12:15:55  #文章生成时间，一般不改，当然也可以任意修改
tags: [Hexo, Ocean] #文章标签，可空。也可以按照你的习惯写分类名字，注意后面有空格，多个标签可以用[]包含，以`,`隔开
categories: [技术] #分类
---
假期宅在家里，研究了一下用github搭建个人网站，把里面使用到的工具和命令总结一下。相关代码可参考：https://github.com/majing2019/myblogs
<!--more-->

# 安装相关软件
>参考：
>https://zhuanlan.zhihu.com/p/26625249
>https://zhuanlan.zhihu.com/p/62555815


* npm install -g hexo-cli
* hexo init blog
* cd ~/blog
* export CC=/usr/bin/clang
* export CXX=/usr/bin/clang++
* npm install
* npm install hexo-server --save
* hexo server
* 在[http://localhost:4000](http://localhost:4000)访问网站首页
* npm install hexo-deployer-git --save



# 建立repository

>参考：
>https://help.github.com/en/github/working-with-github-pages
>https://github.community/t5/GitHub-Pages/404-Error/td-p/14331


* 创建username.github.io的repository
* 在Settings->Github Pages中升级账户



# 修改相关配置

* 修改_config.yml
```
deploy:
  type: git
  repo: <repository url> #git@github.com:sufaith/sufaith.github.io.git
  branch: [branch] #master
  message: [message]
url: majing2019.github.io
```
* 在source文件夹下创建CNAME文件，内容为二级域名
* 在~/blog目录下运行hexo generate
* hexo clean && hexo deploy
* 访问 [https://majing2019.github.io/archives/](https://majing2019.github.io/archives/)



# 绑定域名

* 登录[https://www.aliyun.com/](https://www.aliyun.com/)注册了一个域名majsunflower.cn
* 添加一个域名解析
  * 类型CNAME，主机记录www，记录值majing2019.github.io
  * 类型A，主机记录@，记录值是对应的ip地址，可通过ping majing2019.github.io获得
* 在github仓库中设置custom domain
* 在blog下创建source/CNAME文件，并写入majsunflower.cn



# 编写自己的个性化网站

>参考：
>https://zhwangart.github.io/2018/11/30/Ocean/




## 更换主题

* 在[https://hexo.io/themes/](https://hexo.io/themes/)中选择一个主题
* git clone [https://github.com/zhwangart/hexo-theme-ocean.git](https://github.com/zhwangart/hexo-theme-ocean.git) themes/ocean
* 修改_config.yml中theme为ocean



## 配置语言

* _config.yml中language改为zh-CN



## 评论功能

>参考：
>https://zhwangart.github.io/2018/12/06/Gitalk/


* 在[https://github.com/settings/applications/new](https://github.com/settings/applications/new)申请
  * 后续可在[https://github.com/settings/developers](https://github.com/settings/developers)中修改app相关内容
  * 注意Authorization callback URL在网站绑定域名后需要写域名
* 填写themes/ocean/_config.yml中gitalk相关字段



## 使用图床

>参考：
>https://zhuanlan.zhihu.com/p/26625249
>https://blog.csdn.net/qq_36305327/article/details/71578290


* 到[https://www.qiniu.com/](https://www.qiniu.com/)上添加对象存储[https://portal.qiniu.com/kodo/bucket/](https://portal.qiniu.com/kodo/bucket/)
* 在markdown中可直接引用图片
```
![sunflower](http://q503tsu73.bkt.clouddn.com/IMG_3012.JPG?e=1580527998&token=05Ii263bPN3Z-CT3JPRaRfWi5sXIj8pwX6V1bN2j:6FFnse-_gOPpSeTWpN-i9hJ1pwQ=&attname=)
```


## 添加关于

* hexo new page about
* 使用markdown编写source/about/index.md



## 添加标签

* hexo new page tags // 创建标签页面
* 修改source/tags/index.md为
```
---
title: Tags
date: 2019-04-19 17:28:54
type: tags
layout: "tags"
---
```
[注]: 目前oceans主题还不支持标签


## 添加相册

* hexo new page gallery
* 编辑source/gallery/index.md
```
---
title: Gallery
albums: [
        ["img_url","img_caption"],
        ["img_url","img_caption"]
        ]
---
```
* 如果出现相册加载过慢的问题，可以参考[https://zhwangart.github.io/2019/07/02/Ocean-Issues/](https://zhwangart.github.io/2019/07/02/Ocean-Issues/)解决



## 添加分类

* hexo new page categories

[注]: 目前oceans主题还不支持分类



## 本地搜索

>参考：
>https://github.com/zhwangart/gitalk/issues/7#issuecomment-451877736


* npm install hexo-generator-searchdb --save
* 在blog/_config.yml中添加配置
```
search:
  path: search.xml
  field: post
  content: true
```
* hexo g
* ~~修改themes/ocean/layout/_partial/after-footer.ejs中修改如下内容~~
```
<% if (theme.local_search.enable){ %>
  <%- js('/js/search') %>
<% } %>

<%- js('/js/ocean') %>
```


## 用Markdown写文章



### 创建文章

>参考：
>https://chaxiaoniu.oschina.io/2017/07/10/Markdown-Grammar/


* hexo new "用GitHub+Hexo搭建个人网站"
* 文章格式如下
```
---
title: 用GitHub+Hexo搭建个人网站 #文章页面上的显示名称，可以任意修改
date: date  #文章生成时间，一般不改，当然也可以任意修改
tags: [Hexo, Ocean] #文章标签，可空。也可以按照你的习惯写分类名字，注意后面有空格，多个标签可以用[]包含，以`,`隔开
categories: [技术] #分类
---
这里是你博客列表显示的摘要文字
<!--more-->
以下是博客的正文，以上面的格式为分隔线
```
* 如果不希望显示时有目录，需要添加
```
toc: false
```


### 添加公式

>参考：
>https://blog.csdn.net/Aoman_Hao/article/details/81381507


* npm uninstall hexo-renderer-marked --save
* npm install hexo-renderer-kramed --save
* 修改node_modules/hexo-renderer-kramed/lib/renderer.js
```
function formatText(text) {
  // Fit kramed's rule: $$ + \1 + $$
  // return text.replace(/`\$(.*?)\$`/g, '$$$$$1$$$$');
  return text;}
```
* npm uninstall hexo-math --save

* npm install hexo-renderer-mathjax --save
* 修改node_modules/hexo-renderer-mathjax/mathjax.html，注释掉最后一行script并改为
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```
* 修改node_modules/kramed/lib/rules/inline.js
```
escape: /^\\([`*\[\]()# +\-.!_>])/,
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```
* 修改themes/ocean/_config.yml增加
```
mathjax: true
```


### 添加文章封面

```
---
title: Post name
photos: [
        ["img_url"],
        ["img_url"]
        ]
---
```


### 添加视频

>参考：
>https://blog.csdn.net/u010953692/article/details/79075884
```
<iframe height=498 width=510 src="http://q503tsu73.bkt.clouddn.com/IMG_0018.mp4?e=1580557032&token=05Ii263bPN3Z-CT3JPRaRfWi5sXIj8pwX6V1bN2j:-rUb7zOxk-WfRrhdJtNdOOGfy58=&attname=" frameborder=0 allowfullscreen></iframe>
```


### 文章置顶

* npm uninstall hexo-generator-index --save
* npm install hexo-generator-index-pin-top --save
* 在需要置顶的文章上加入
```
---
 title: 新增文章置顶
 top: ture
 ---
```


# 同时部署在Github和Coding上

>参考：
>https://tomatoro.cn/archives/3de92cb5.html


* [https://coding.net/](https://coding.net/)上创建devops项目
* 修改blog/_config.yml中的deploy
```
deploy:
  type: git
  repo: 
    github: git@github.com:majing2019/majing2019.github.io.git
    coding: git@e.coding.net:majsunflower/myblog.git
  branch: master
  message: my blog
```
* 将id_rsa.pub的公钥复制到个人账户下，ssh -T git@git.coding.net验证是否成功
* hexo deploy -g部署到coding上
* 配置静态页面即可访问：[https://02ss3u.coding-pages.com/](http://02ss3u.coding-pages.com/)
* 在自定义域名里增加：[majsunflower.cn](http://majsunflower.cn/)
* 阿里云[https://homenew.console.aliyun.com/](https://homenew.console.aliyun.com/)中修改域名相关配置，区分境内和境外的访问



# 常用命令

* 部署：hexo clean && hexo g && hexo d

* 本地测试：hexo server
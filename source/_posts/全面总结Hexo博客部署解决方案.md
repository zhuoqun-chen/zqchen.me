---
title: 全面总结Hexo博客部署解决方案
date: 2020-11-26 13:59:46
categories: uncategorized
tags: [Hexo,Git,Github Pages,Github Actions]
urlname: an-overall-summary-of-solutions-to-hexo-blog-deployment
---
# 前言

+ 在我这次搭建博客的过程中，学到了很多新东西，查找网页教程的过程中看到很多大佬的博客，在浏览他们博文的同时，了解了一些新的概念和名词，非常感谢～

+ 整个搭建的方案经历了几次转折，我最初在什么都不懂的情况下使用的是 Hexo 提供的一键部署方案，最后采取了自动化部署的方案。

+ 欢迎到 [俺の小屋](https://zqchen.me) 观光 []~(￣▽￣)~* 乾杯~~~



# 概述




## 两种部署方式

+ 小白版：直接利用 hexo 框架的 hexo init && hexo clean && hexo deploy

+ 进阶版：利用 Github Actions 实现自动化部署（hexo generate 生成的 ./public 文件夹直接生成在 Github 托管的虚拟机上，本地调试的时候可以用 hexo server）   

  

## 主题选择

搭建过程中接触过的几个主题：

+ [hexo-theme-next](https://github.com/iissnan/hexo-theme-next)
+ [hexo-theme-geek](https://github.com/zhuoqun-chen/hexo-theme-geek)
+ [hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)
+ [hexo-theme-butterfly](https://github.com/jerryc127/hexo-theme-butterfly)



## 字体选择

+ [Zpix](https://github.com/SolidZORO/zpix-pixel-font)

+ [xkcd-font](https://github.com/ipython/xkcd-font)



## 知识要点

+ Git 基础命令（fork / clone / add / commit / push / fetch / merge / update / remote add / status / log / pull） 和 Git 高级用法（submodule）以及 Git 工具（gitk / git-gui）
+ Github 提供的几项服务：[Github Pages](https://pages.github.com/) / [Github Actions](https://github.com/features/actions)
+ 域名 / IP地址 / DNS解析 三者之间的关系、A记录 / CNAME记录 / NS记录的编写
+ 网页编程： HTML / CSS / JavaScript / Node.js（包管理工具：npm） / [EJS](https://ejs.bootcss.com/) 框架 / [YAML](https://yaml.org/)（.yml文件）
+ Shell / Bash 脚本（echo 输出 / sed 工具）
+ CDN 加速（[jsdelivr](https://www.jsdelivr.com/)）



## 搭建前的准备

+ 一个域名：[zqchen.me](https://zqchen.me)（注册商是 [GoDaddy](https://sg.godaddy.com/zh)，目前给我提供 DNS解析服务的也是这家注册商，由于最终解析到的服务器属于Github，架设在美国，故解析不需要国内备案）
+ 搞清楚 DNS服务提供商要求你输入的A记录、CNAME记录、NS记录是什么；如何填写这几个记录，使得在浏览器上输入我们的域名敲一个回车就可以直接访问到我们托管在 Github Pages 上的页面



# 小白版方案

Github Pages 官网介绍搭建的 Tutorial + Hexo 官网参考手册在 Github Pages 上的一键部署方案（hexo init && hexo clean && hexo generate && hexo server 后在 默认的 localhost:4000 上查看本地未部署的静态页面的解析效果，这个效果和最终发布在 Github Pages上联网访问的效果是一致的）



## 最初做法

最初我的方法是这样的，这也是网上大多数教程的方法：hexo 框架生成渲染后的静态页面，将此页面通过 hexo 提供的命令hexo deploy 一键部署到 Github Pages 个人主页：zhuoqun-chen.github.io 上，并且修改 setting，使得 https://zhuoqun-chen.github.io/ 可以使用新的域名 https://zqchen.me/，接下来在 DNS解析 那里添加1个 CNAME 记录（zqchen.me --> zhuoqun-chen.github.io）和4个A记录（zqchen --> 185.162.118.10），就可以了。（参考资料：[配置 GitHub Pages 站点的自定义域](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site)）



## 改进做法

后来我觉得把博客直接搭建在 https://zhuoqun-chen.github.io/ 下有点不妥，因为我想留着这个域名以后发布一些简历或者个人介绍什么的，那么就想把博客项目单独新建一个仓库 zqchen.me，在这个仓库里设置 main 分支为 Github Page。



事实上，Github Pages 服务分为两种：

1. 个人 或 组织及企业 账户的主页
2. 每个项目的展示页面，也即每个仓库的展示页面

Github 给每个账号提供的域名就是 username.github.io，情况1就要求用户新建一个仓库名必须是 username.github.io，这个仓库建好后在 setting 里开启 Github Pages 服务，默认的域名就是 https://username.github.io

对于情况2，仓库名可以随便起，但是设定该仓库的某个分支为 Github Pages 后，默认的域名是 https://username.github.io/repo/

Github 还要求对项目仓库自定义域名时必须在 CNAME记录中 让域名指向 username.github.io，而不能是 username.github.io/repo， 同时要在setting里面写入自定义域名，假如这个域名被别人的仓库给写到他的setting里了，Github就不允许你写入你的setting里了（这是一个坑~ 在[GitHub Pages 官方文档](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/getting-started-with-github-pages)里有写到）



## 缺点及弊端

+ 想要更新博客必须在同一台本地电脑上，一旦电脑丢失或损坏个人博客也就不复存在了（因为 hexo deploy 只是把 public 文件夹下的所有内容给覆写到 zhuoqun-chen/zhuoqun-chen.github.io 仓库的唯一的 main 分支上，Github 上并不存在博客的主题配置文件、各个 .md 源文件等，而仅有渲染后的各个静态文件 html、脚本文件 .js、图片 .jpg/.png。）
+ 因为缺少搭建环境，无法在移动端（手机、iPad）或者其他电脑上随心所欲的写博文并上传让其他人看到。



# 进阶版方案

在参考了很多教程，对这一套流程都很熟悉了之后，我采取了 [journey-ad](https://github.com/journey-ad) 大佬现在的这个方案：生成站点内容的源文件 同 站点文件本身分离，分别来维护，后者的生成并上传采用的是 Github Actions 服务。



## 方案架构

+ 内容、主题、配置文件、图片等单独维护，由 Hexo 生成的站点文件单独维护

+ 博客使用的主题设置成子模块：在本地只用 hexo init 生成必要的几个文件和文件夹，而部署就不采用 hexo 的一键部署了，接着在 themes/ 下引入主题文件夹 hexo-theme-geek 作为这个博客仓库的子模块 submodule，在本地使用还需 git submodule init && git submodule update 等等

+ 博客项目单独建立一个仓库名 zqchen.me，并创建两个分支
  + main 分支包含：内容及模板（scaffolds 和 source 文件夹）、站点配置文件（_config.yml）、主题（themes 文件夹）、其他（package.json）等资源
  + gh-pages 分支则包含：由 GitHub Actions 自动化生成的静态文件（即对应本地执行 hexo generate 生成的 public 文件夹的所有内容，实际上一系列 actions 后的目标就是在 GitHub 托管的虚拟机上运行 hexo init、hexo genreate 操作，并把在其上生成的 public 文件夹以及我们自己写的 CNAME 文件重新上传到 zqchen.me 仓库的 gh-paghes 分支上，而能够引起一系列动作的源头，我们在 .github/workflow/deploy.yml 中设定为 main 分支有了新的 push ，即每当我们执行 git push origin main 时，就会自动引发 deploy.yml 所指定的 Github Actions 动作流。）



## 具体实施

+ Github新建远程仓库 zqchen.me，在默认分支 main 下新建一个 gh-pages 分支，在 setting 里设置 gh-pages 分支使用 Github Pages 服务

+ 本地电脑 hexo init，生成 node_modules/ | scaffolds/ | themes/ | source/ | package.json | _config.yml
+ 在 themes/ 文件夹下引入 submodule 主题 hexo-theme-geek，此时会多出来 .gitsubmodule 和 themes/hexo-theme-geek/ 文件夹
+ 修改站点配置文件 _config.yml：修改基本信息 | 取消使用 highlight | 主题 landscape --> hexo-theme-geek
+  新建 .github/workflows/deploy.yml 动作流指令文件，编写相应的自动化部署操作流程
+ 将以上涉及到的全部文件和文件夹刨除 node_modules/ 之外全部 git commit && git push origin main，由于 push 操作引发了 deploy.yml，会自动执行一系列操作，站点静态页面和 CNAME文件最终被远端自动上传到 gh-pages 分支
+ 在浏览器中输入 [zqchen.me](https://zqchen.me)，联网访问成功



## 方案优势

+ 速度快：git push 到远端仓库后，执行动作流的虚拟机和仓库服务器本身都在美国，所以远端之间上传和下载的速度都很快，真正做到了 git push  “一键部署”

+ 可以在手机或者iPad设备上登录网页版的 Github，直接新建一个 .md 文件并撰写内容，commit 之后，重新访问网站后就可以看到加入新博文的网站了，无需电脑的参与。最大的优势在于，即使电脑损毁，生成博客所需的所有内容依然存储在远端，随时可以执行 git clone 后恢复。



## 缺点及弊端

但是这个方案有一个缺点，就是单独更新 main 分支但更新的内容并不是 markdown 文章，例如只是更新一下 README.md，commit 并 push 之后仍然会启动 actions 工作流重新生成一次同样的站点文件，虽然说由于前后的 public 文件夹一模一样，执行peaceiris/actions-gh-pages@v3 后并不会把虚拟机上的 public 覆写到 gh-pages 分支上，但这样做依然会浪费远端虚拟机的一些资源。（当然对于使用者的我们而言还是很方便的b（￣▽￣）d）



# 结语

欢迎到[俺の小屋](https://zqchen.me)观光 []~(￣▽￣)~* 



## 用到的 Git 命令

+ git fetch + git merge
+ git update
+ git submodule add 
+ git status
+ git add .
+ git checkout
+ git commit -a / -m
+ git revert 版本回退



## 用到的Github Actions

github_token 权限的设置以及把 token 添加到某个仓库中

+ actions/checkout@master --> actions/checkout@v2

+ peaceiris/actions-gh-pages@v3

+ actions/create-release@v1

  

## 一些新玩意儿

+ ruby / SpingBoot / rail

+ Cloudflare

+ xml / svg 图片

+ Web 字体简介: TTF, OTF, WOFF, EOT & SVG

+ Typescript

+ 图床：Booru Project 及其衍生的 Danbooru、[Moebooru](https://wiki.bibanon.org/Moebooru) 等等（[Booru是什么？](https://tieba.baidu.com/p/5284041707?red_tag=1142919937)）



## NSFW网站（逃）

+ rule34.xxx
+ gelbooru.com



## 图标网站

+ 提供图标的网站 [Icons8](https://icons8.com/icons/set/star)

+ 制作自己的 favicon（[favicon制作-在线工具](https://tool.lu/favicon/)）



# 参考来源



## 官方参考

+ [Git - Book](https://git-scm.com/book/zh/v2)
+ [Git - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
+ [Hexo 官方文档](https://hexo.io/zh-cn/docs/) ---- Hexo官方参考手册里的变量和辅助函数需要仔细看看，因为主题源代码里写有很多这样的语句和变量，最开始看代码的时候我根本看不懂。
+ [GitHub Pages 官方文档](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/getting-started-with-github-pages)
+ [GitHub Actions 官方文档](https://docs.github.com/cn/free-pro-team@latest/actions)



## 大佬的博客

+ [DIYgod](https://diygod.me/)
+ [jouney-ad](https://github.com/journey-ad)（live2D看板娘、CPlayer和DPlayer播放器）
+ [阮一峰的网络日志](http://www.ruanyifeng.com/blog/)（[YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)、[GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)）



## Hexo 部署方案

+ [使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762/answer/489124966)----简写版：[两分支部署Hexo](https://www.cnblogs.com/oeong/p/12326624.html) ，此方案的梳理：[hexo博客同步管理及迁移](https://www.jianshu.com/p/fceaf373d797) 说明：这个方案没有用到 Github Actions，但总的来说和我目前的方案是一致的，同属于“一个仓库的两个分支分别负责内容和站点”方案
+ [Hexo+GitHub Actions 完美打造个人博客](https://blog.csdn.net/z_cheng_xiao/article/details/106055153)
+ [Hexo：语雀云端写作，Github Actions持续集成](https://blog.csdn.net/z_johnny/article/details/104629805?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control) ---- 涉及到Github_TOKENS的使用
+ [如何将你的github仓库部署到github pages（github.io博客）](https://www.cnblogs.com/wanliyuan/p/5673622.html)
+ [Hexo 使用 Github Actions 自动更新 - 简书](https://www.jianshu.com/p/7940fe40885d)



## Git 命令 / Github Pages / Github Actions

+ [Git - Book 官方文档]()

+ [Git基础教程 - 简书](https://www.jianshu.com/p/bb8acd3dae43) ---- 很详细，很全面的总结

+ [Git Fork后与源作者同步更新](https://blog.csdn.net/yuanlaijike/article/details/83722163)

+ [github上fork了别人的项目后，再同步更新别人的提交](https://blog.csdn.net/qq1332479771/article/details/56087333?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control)

+ [git fork之后的项目如何保持和上游同步](https://blog.csdn.net/happyfreeangel/article/details/102605329?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control)

+ [Git 怎样保证fork出来的project和原project（上游项目）同步更新](https://blog.csdn.net/Jason_Fish/article/details/65939081?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

+ [github fork项目更改后与原作者同步更新](https://www.cnblogs.com/guors/p/10938035.html)

+ [怎么把自己fork别人的仓库中的代码更新至最新版本？](https://segmentfault.com/q/1010000009818126)

+ [github 自己创建了一个项目A，我的同事fork一个B，当我的项目更新的时候，怎么样在他fork的repo上进行相应的更新?](https://www.zhihu.com/question/20171506)

+ [解决原仓库和fork仓库分支问题](https://blog.csdn.net/whldeBK/article/details/103883467)

+ [深入浅出 Git 权限校验_diancheng8834的博客-CSDN博客](https://blog.csdn.net/diancheng8834/article/details/101943626?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduend~default-1-101943626.nonecase&utm_term=github%20token%E6%80%8E%E4%B9%88%E8%AE%BE%E7%BD%AE%E6%9D%83%E9%99%90&spm=1000.2123.3001.4430)

+ [git fetch & pull详解](https://www.cnblogs.com/runnerjack/p/9342362.html)

+ [git fetch -- 简书](https://www.jianshu.com/p/d07f5a8f604d)

+ [git fetch 命令 -- 菜鸟教程](https://www.runoob.com/git/git-fetch.html)

+ [git diff 命令 -- 菜鸟教程](https://www.runoob.com/git/git-diff.html)

+ [git commit -a 命令困惑 · Ruby China](https://ruby-china.org/topics/4030)

+ [Git语法之Checkout使用 - 简书](https://www.jianshu.com/p/37f3a7e4a193)

+ [Git - 子模块 官方文档](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

+ [git submodule 的使用 - 简书](https://www.jianshu.com/p/e27a978ddb88)

+ [git中submodule子模块的添加、使用和删除_guotianqing的博客-CSDN博客_git submodule](https://blog.csdn.net/guotianqing/article/details/82391665)

+ [Git Submodule使用完整教程 - lsgxeva - 博客园](https://www.cnblogs.com/lsgxeva/p/8540758.html) ---- 强烈推荐

+ [Git中submodule的使用 - 知乎](https://zhuanlan.zhihu.com/p/87053283)

+ [Git Submodule简单使用_巫山老妖-CSDN博客_git submodule](https://blog.csdn.net/wwj_748/article/details/73991862?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1.control)

+ [使用Git Submodule管理子模块 - 简书](https://www.jianshu.com/p/b49741cb1347)

+ [git-github 子模块仓库更新（git submodule）/git中submodule子模块的添加、使用和删除_西京刀客-CSDN博客](https://blog.csdn.net/inthat/article/details/108416238)

+ [git submodule 添加，更新与删除_qq_41163331的博客-CSDN博客](https://blog.csdn.net/qq_41163331/article/details/81672836?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduend~default-1-81672836.nonecase&utm_term=git%20%E4%B8%BB%E9%A1%B9%E7%9B%AE%20%E5%AD%90%E6%A8%A1%E5%9D%97%E6%9B%B4%E6%96%B0%E5%90%8E&spm=1000.2123.3001.4430)

+ [GitHub进行版本回退_Java识堂的博客-CSDN博客_github 回滚](https://blog.csdn.net/zzti_erlie/article/details/87189530)

+ [Git恢复之前版本的两种方法reset、revert（图文详解）_游笑天涯-CSDN博客_git revert](https://blog.csdn.net/yxlshk/article/details/79944535?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control)

+ [GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

+ [GitHub Actions 官方文档](https://docs.github.com/cn/free-pro-team@latest/actions)

+ [GitHub Actions详解](https://blog.csdn.net/pythonxiaopeng/article/details/108869728)

+ [GitHub 工作流 Actions 的使用 以前端项目为例](https://fizzz.blog.csdn.net/article/details/107912095)

+ [Github Actions简单部署一个vue/react项目 - 陌上兮月 - 博客园](https://www.cnblogs.com/zhangnan35/p/13272638.html)

+ [GitHub-Actions的使用教程_RopeHuo-CSDN博客](https://blog.csdn.net/qq_24464827/article/details/109836210)

+ [actions自动化：vuepress构建，github同步gitee，giteepage部署 - 知乎](https://zhuanlan.zhihu.com/p/302530881)

+ [使用GitHub Actions 工作流 自动创建一个release_知无涯-CSDN博客](https://blog.csdn.net/github_35631540/article/details/107914101)

+ [github获取token](https://blog.csdn.net/u014175572/article/details/55510825)

+ [设置 GitHub Token](https://www.jianshu.com/p/32e238216a4d)

+ [GitHub创建token](https://blog.csdn.net/weixin_44662563/article/details/105862622)

+ [GitHub Pages 官方文档](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/getting-started-with-github-pages)

+ [用Github Pages展示你的项目 - 简书](https://www.jianshu.com/p/068c349d5949?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

+ [关于同一个github账号下两个github pages项目仓库域名跳转的问题。。。 - 的回答 - SegmentFault 思否](https://segmentfault.com/q/1010000006213476/a-1020000006214088)

  

## JS / Node.js

+ [node 使用utility实现字符串加密](https://blog.csdn.net/weixin_41111068/article/details/87925232?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control)
+ [md5加密依赖工具utility使用小记](https://blog.csdn.net/weixin_39786582/article/details/88117185)
+ [ECMAScript 6 简明教程](https://www.runoob.com/w3cnote/es6-concise-tutorial.html)
+ [js中let和var定义变量的区别](https://blog.csdn.net/nfer_zhuang/article/details/48781671)
+ [JavaScript let 和 const ----ECMAScript 2015(ECMAScript 6)](https://www.runoob.com/js/js-let-const.html)
+ [nodejs中 require 方法的加载规则](https://blog.csdn.net/qq_36410795/article/details/86484286)
+ [安装包的方式(require, install, update)对比](https://www.cnblogs.com/iceman-/p/8574218.html)



## npm

+ [node 使用utility实现字符串加密](https://blog.csdn.net/weixin_41111068/article/details/87925232?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control)
+ [npm 查看安装了哪些包](https://blog.csdn.net/palmer_kai/article/details/79723907)
+ [关于npm安装全局模块，require时报Error: Cannot find module 'XXX'的解决办法](https://blog.csdn.net/zhuyin365/article/details/88423438)
+ [Module build failed: Error: Cannot find module 'node-sass' Require stack报错解决](https://blog.csdn.net/correlate/article/details/102768816)
+ [npm全局安装和局部文件安装区别](https://www.cnblogs.com/linziwei/p/7786895.html)
+ [npm 全局安装和局部安装的区别](https://www.jianshu.com/p/e839326ce30d)
+ [npm和yarn的区别和对比](https://blog.csdn.net/qq_27674439/article/details/93976115)
+ [Yarn 概念 | Hexo](http://chentyit.cn/2019/10/10/Yarn-%E6%A6%82%E5%BF%B5/#%E9%87%8D%E8%A6%81%E6%A6%82%E5%BF%B5)



## HTML

+ [div里常用的class命名](https://www.cnblogs.com/9303lei/p/5432628.html)



## Linux

+ [Linux sed 命令 | 菜鸟教程](https://www.runoob.com/linux/linux-comm-sed.html)
+ [sed -i 命令解释 - hxing - 博客园](https://www.cnblogs.com/hxing/p/12008354.html)
+ [Openssl rand命令_weixin_30564901的博客-CSDN博客](https://blog.csdn.net/weixin_30564901/article/details/99469472)
+ [sed -i命令详解 - 君子笑而不语 - 博客园](https://www.cnblogs.com/xiaoliu66007/p/11059527.html)
+ [sed_一直被模仿，从未被超越-CSDN博客](https://blog.csdn.net/mshxuyi/article/details/103233628?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.control)



## Hexo博客基础配置

+ [Hexo 官方文档](https://hexo.io/zh-cn/docs/)
+ [GitHub Pages 官方文档](https://docs.github.com/cn/free-pro-team@latest/github/working-with-github-pages/getting-started-with-github-pages)
+ [GitHub Page+Hexo+nexT 搭建个人博客](https://zhuanlan.zhihu.com/p/46096258)
+ [利用 Github+Hexo 搭建个人博客网站](https://zhuanlan.zhihu.com/p/98427357)
+ [5分钟搞定个人博客-hexo](https://www.jianshu.com/p/390f202c5b0e)
+ [Hexo使用指南](https://www.jianshu.com/p/84a8384be1ae?appinstall=0)
+ [域名 DNS 解析到自己的 Github Page 页面（github page 绑定自己的域名）](https://www.cnblogs.com/xieqk/p/Github-Page-DNS.html)
+ [Github Pages 绑定域名遇到的坑](https://blog.csdn.net/i_do_not_know_you/article/details/105594269)
+ [Hexo常用命令](https://www.jianshu.com/p/7b8faf77d1af)
+ [用Hexo搭建博客（二）——修改基本内容](https://blog.csdn.net/qq_35117024/article/details/81390780)
+ [Hexo 站点配置文件内容解释](https://blog.csdn.net/lijing742180/article/details/85722664)
+ [Hexo全局配置文件config.yml详解](https://zhuanlan.zhihu.com/p/127786638)
+ [Hexo+NexT（二）：Hexo站点配置详解](https://blog.csdn.net/loze/article/details/94209583)
+ [【源码开放】Hexo+Github 博客butterfly 和 matery 主题 搭建完全教程【整理】 | 超逸の博客](https://yangchaoyi.vip/posts/520520/#%E5%89%8D%E8%A8%80) ---- 强烈推荐，炒鸡详细~
+ [Hexo+Github博客搭建完全教程 | 洪卫の博客](https://sunhwee.com/posts/6e8839eb.html) ---- 强烈推荐，炒鸡详细~



## 常见问题

+ [Hexo常见问题解决方案](https://wp.huangshiyang.com/hexo%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)



## 主题和字体

+ [Hexo主题开发](https://www.cnblogs.com/yyhh/p/11058985.html)----强烈推荐，该文章讲的是制作一个完整的主题的全过程，涉及到很多代码方面的东西
+ [hexo博客next主题添加自定义字体comic neue](https://leflacon.top/7167e0bc/)
+ [Hexo 自定义字体](https://milktea.info/2019/01/25/ComicNeue/)
+ [hexo博客的背景设置](https://blog.csdn.net/Com_ma/article/details/76039859)----强烈推荐，添加各种美化的小部件的步骤很齐全
+ [教你定制Hexo的landscape打造自己的主题](https://www.php.cn/div-tutorial-269258.html)
+ [【个人博客搭建系列二】hexo框架中自定义博客页面主题](https://blog.csdn.net/qq_40979624/article/details/105310577?utm_medium=distribute.pc_relevant.none-task-blog-title-3&spm=1001.2101.3001.4242)



## 个性化及性能优化

+ [又折腾了一次，Hexo 真香](https://milktea.info/2019/01/23/begin/#more)
+ [Hexo博客添加自定义HTML页面](https://blog.csdn.net/qq_40922859/article/details/100877777?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)
+ [Hexo 添加自定义的内置标签](https://blog.csdn.net/weixin_30700099/article/details/94807045)
+ [hexo的图标配置](https://www.jianshu.com/p/7519c71c8ea9)
+ [Hexo博客的优化-提升访问速度，SEO](https://www.zyskys.com/posts/60945.html)
+ [Hexo折腾记——性能优化篇](https://blog.csdn.net/weixin_34387468/article/details/90681868)
+ [（收录）Hexo Landscape主题的字体和JS库优化](https://www.jianshu.com/p/ffcdc4fec6ec)
+ [Hexo+Github个人博客搭建美化 - 知乎](https://www.zhihu.com/column/c_1215655729367113728)
+ [对 Hexo 博客文章进行加密 - 知乎](https://zhuanlan.zhihu.com/p/113235573)
+ [Category: Hexo | TRHX'S BLOG](https://www.itrhx.com/categories/Hexo/)
+ [好用的jsDelivr - 木人子韦一日尘 - 博客园](https://www.cnblogs.com/murenziwei/p/12747311.html)
+ [利用jsDelivr白嫖全球超高速静态资源访问服务！ - 哔哩哔哩](https://www.bilibili.com/read/cv4297993)



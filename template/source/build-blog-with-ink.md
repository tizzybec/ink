title: 使用InkPaper搭建个人博客
date: 2015-10-04 11:33:00 +0800
update: 2015-10-04 11:33:00 +0800
author: me
cover: -/images/ink.png
tags:
    - 工具
preview: ink是一款简洁的静态博客构建工具，结合github pages能够快速搭建个人博客

--- 

## 建站缘由

由于在一家涉密机构工作，工作两年一直没有将技术学习过程和成果以博客形式分享出来（一直零散地写在有道笔记里，不成体系）。恰巧最近外甥在学习[React Native](https://facebook.github.io/react-native/),有些不明白的地方在不断请教我，借此机会我也搭建了react-native相关环境，对react/react-native有些比较基本的理解，由于使用过程中难免会有问题，所以想以此博客记录疏漏，同时激励自己将写博客这件事情坚持下去。

之前无意中看到一款不错的静态博客，源自一款流产的产品，使用go语言编写，帮助文档简洁明了，于是就决定作为新博客的构建工具，搭建在github上，同时购置了域名[www.tizzybec.com](http://www.tizzybec.com)定向至github pages.

![纸小墨 - 简洁的静态博客构建工具](-/images/ink.png)

## 搭建过程

### 建立github pages

- 创建一个新的reponsitory，取名`tizzybec.github.io`（tizzybec是我的英文id），
- 进入settings，在Github Pages部分，点击`Launch automatic page generator`覆盖原有网站，否则就不能用`tizzybec.github.io`来访问博客主页
- 进入`tizzybec.github.io`即可看到新生成的github页面

### 搭建本地博客编写环境

#### 参照ink的帮助文档部分直接从源码构建

- 安装go环境`brew install go`，设置GOPATH为`export GOPATH="~/Github/go"`
- 为了方面以后修改，从ink的github页面直接fork一份
- 运行`go get github.com/tizzzybec/ink`获取ink极其依赖并编译，编译后的ink可执行文件位于$GOPATH/bin下
- 添加`expoert=$PATH:$GOPATH/bin`到~/.zshrc文件，以便能随时调用ink命令
- 运行`ink preview $GOPATH/src/github.com/tizzybec/ink/template`后，按控制台提示打开浏览器即可预览
- 切换到工作目录为`$GOPATH/src/github.com/tizzybec/ink/template`，在source目录下编写markdown文件，运行`ink build`生成博客内容，在浏览器预览中能看到实时的排版效果

#### 编辑`config.yml`，参照注释进行常用信息的修改

``` yaml

site:
    title: 生活∙编码-日记
    subtitle: keep it simple and stupid.
    logo: -/images/tizzybec.jpg
    limit: 10
    theme: theme
    disqus: tizzybec
    lang: zh
    url: http://www.tizzybec.com/
    # root: /blog

authors:
    me:
        name: tizzybec
        intro: c++码农，haskell/clojure学习中...
        avatar: -/images/tizzybec.jpg

build:
    port: 8000
    # Copied files to public folder when build
    copy:
        - theme/css
        - theme/js
        - theme/favicon.ico
        - theme/robots.txt
        - source/images
    # Excuted command when use 'ink publish'
    publish: |
        export from_path=~/Github/go/src/github.com/tizzybec/ink/template/public/
        export to_path=~/Github/tizzybec.github.io/
        cp -RP $from_path $to_path
        cd $to_path
        git add . -A
        git commit -m "update"
        git push origin

```

#### 添加cc协议标识

修改_footer.html文件
``` html
<span class="license">
      <a rel="license" href="//creativecommons.org/licenses/by/4.0/" title="Creative Commons Attribution 4.0 International license">
        <img src="{{.Site.Root}}/images/cc88x31.png" alt="License" data-pin-nopin="true" />
      </a>
</span>
```
修改index.less文件

``` css
.license {
      float: right;
      padding: 0;
}
```

### 绑定自定义域名

去 *万网/dnspod/花生壳* 都可以申请到域名，付费后到控制台新建一条域名解析记录

```
子域名 记录类型   记录值                TTL
www   CNAME     tizzybec.github.io  600
```

在`tizzybec.github.io`仓库根目录执行`echo "www.tizzybec.com" > CNAME`，提交到github

现在访问[tizzybec](http://www.tizzybec.com/)就是最终的个人博客站点了。

## 其它

- 修改了coffee/less文件需要使用gulp重新构建前端文（`cd theme && gulp && cd .. && ink build`）
- 暂时不支持单个md的构建命令，后续如果支持比较好


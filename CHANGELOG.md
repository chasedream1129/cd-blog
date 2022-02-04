---
layout: plain
sitemap: false
---

# CHANGELOG
{:.no_toc}

* this list will be replaced by the toc
{:toc .large-only}
## v0.1.0(Init)
2022.02.04
{:.heading.post-date}

### 初始化

* 基于Hydejack PRO v9.1.5的"starter-kit-gh-pages"搭建。

### 自定义
* "sidebar-bg.jpg"背景图
    
    替换为旗原老师绘制的CD子。

* "icons"文件夹中的图标
    
    替换为旗原老师原作，我照着点出来的像素CD子。

* 社交媒体
    
    添加Mastodon, Telegram, Github, Twitter, E-Mail, Steam

    (默认的Icomoon里没有Mastodon图标，自己加了一个。)

* logo.png
    
    替换为自己的主头像。

* 网站配色
    
    替换强调色为rgb(180,131,124)

    替换主题色为rgb(71,25,26)

    (其实不太满意，之后肯定会换2333)

### 优化

* 上一页与下一页

    不知道是奥地利的习惯还是啥，Hydejack的翻页默认布局是左“Newer“右“Older“，对中国用户来说实在是反人类。改下“_includes/components/pagination.html“就行。

* PWA图标遮罩

    PWA所依赖的manifest “assets/site.webmanifest“ 中，Hydejack将"purpose"设为"any maskable"会导致Chrome自动给透明图标加上黑底遮罩，删掉maskable就行。

    (其实Chrome自己也不建议这么设，开发者工具里面有警告的2333)
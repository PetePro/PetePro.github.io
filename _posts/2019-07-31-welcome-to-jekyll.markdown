---
layout: post
title:  "Welcome to Jekyll!"
crawlertitle: "Jekyll常用方法"
summary: "Welcome to Jekyll!"
date:   2019-07-31 10:00:00 +0800
categories: posts
tags: '理论'
author: xusc
---

你会发现这篇文章在目录 `_posts` 下，如果要添加新的文章，只要在目录 `_posts` 中按照 `YYYY-MM-DD-name-of-post.ext` 格式新建 `.markdown` 文件并引入必要前缀即可。

## markdown 

#### markdown 标题与字体

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题 

粗体：*文字*
斜体：**文字**
```

#### markdown 列表与表格

```
无序列表：
- 文本1
- 文本2
- 文本3

有序列表：
1. 文本1
2. 文本2
3. 文本3

表格：
col1 | col2 | col3
-----|------|-----
a1   | b1   | c1
a2   | b2   | c2
a3   | b3   | c3
```

#### markdown 引用

```
引用链接：
[显示文本](链接地址)

引用图片：
![](图片链接地址)

引用别人文字：
> 引用的文字
```

## Jekyll

#### Jekyll 目录结构

> _posts      博客内容
> _pages      其他需要生成的网页，如About页
> _layouts    网页排版模板
> _includes   被模板包含的HTML片段，可在_config.yml中修改位置
> assets      辅助资源 css布局 js脚本 图片等
> _data       动态数据
> _sites      最终生成的静态网页
> _config.yml 网站的一些配置信息
> index.html  网站的入口

#### Jekyll 常用语法


查阅 [Jekyll docs][jekyll-docs] 了解更多信息，Jekyll的Github的主页： [Jekyll’s Github repo][jekyll-gh] ，如有问题可访问 [Jekyll Talk][jekyll-talk] 进行讨论。

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

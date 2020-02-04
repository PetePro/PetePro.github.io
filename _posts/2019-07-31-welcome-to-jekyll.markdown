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

这篇文章在目录 `_posts` 下，如果要添加新的文章，只要在目录 `_posts` 中按照 `YYYY-MM-DD-name-of-post.ext` 格式新建 `.markdown` 文件并引入必要前缀即可。

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

```
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site
├── .jekyll-metadata
└── index.html
```

文件/目录 | 描述
-|-
_config.yml | 保存配置数据。
_drafts | 未发布的文章。这些文件的格式中都没有 title.MARKUP 数据。

#### Jekyll 常用语法


查阅 [Jekyll docs][jekyll-docs] 了解更多信息，Jekyll的Github的主页： [Jekyll’s Github repo][jekyll-gh] ，如有问题可访问 [Jekyll Talk][jekyll-talk] 进行讨论。

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

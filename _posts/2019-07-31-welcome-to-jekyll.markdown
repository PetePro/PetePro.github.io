---
layout: post
title:  "【Tools】Jekyll & Markdown"
crawlertitle: "Jekyll & Markdown"
summary: "Jekyll & Markdown"
date:   2019-07-31 10:00:00 +0800
categories: posts
tags: 'CSSE'
author: xusc
bg: "SE.jpg"
---

这篇文章在目录 `_posts` 下，如果要添加新的文章，只要在目录 `_posts` 中按照 `YYYY-MM-DD-name-of-post.ext` 格式新建 `.markdown` 文件并引入必要前缀即可。

### Jekyll

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
_includes | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用 include 标签来把文件 _includes/file.ext 包含进来。
_posts | 这里放的就是你的文章了。文件格式很重要，必须要符合: YYYY-MM-DD-name-of-post.ext。
_data | 格式化好的网站数据应放在这里。jekyll 的引擎会自动加载在该目录下所有的 yaml 文件（后缀是 .yml, .yaml, .json 或者 .csv ）。这些文件可以经由 ｀site.data｀ 访问。如果有一个 members.yml 文件在该目录下，你就可以通过 site.data.members 获取该文件的内容。
index.html and other HTML, Markdown, Textile files | 如果这些文件中包含 YAML 头信息 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 .html, .markdown, .md, 或者 .textile 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。

#### Jekyll 常用语法

Jekyll 会遍历网站搜寻要处理的文件。任何有 YAML 头信息的文件都是要处理的对象。对于每一个这样的文件，Jekyll 都会通过 Liquid 模板工具来生成一系列的数据。[Jekyll 常用变量](http://jekyllcn.com/docs/variables/)

[集合（Collections）](http://jekyllcn.com/docs/collections/) 允许定义一种新的文档类型，它既可以像页面和文章那样工作，也可以拥有它们特有的属性和命名空间。

除了内建变量之外，还可以指定用于 Liquid 模板系统的自定义[数据](http://jekyllcn.com/docs/datafiles/)。Jekyll 支持从 _data 目录下的 YAML、JSON 和 CSV 载入数据，注意 CSV 文件必须包含表头行。



### markdown 

#### markdown 头信息

变量 | 描述
-|-
layout | 如果设置的话，会指定使用该模板文件。指定模板文件时候不需要文件扩展名。模板文件必须放在 _layouts 目录下。
permalink | 如果你需要让你发布的博客的 URL 地址不同于默认值 /year/month/day/title.html，那你就设置这个变量，然后变量值就会作为最终的 URL 地址。
published | 如果你不想在站点生成后展示某篇特定的博文，那么就设置（该博文的）该变量为 false。
date | 这里的日期会覆盖文章名字中的日期。这样就可以用来保障文章排序的正确。日期的具体格式为YYYY-MM-DD HH:MM:SS +/-TTTT；时，分，秒和时区都是可选的。
category/categories | 除过将博客文章放在某个文件夹下面外，你还可以指定博客的一个或者多个分类属性。这样当你的站点生成后，这些文章就可以根据这些分类来阅读。categories 可以通过 YAML list，或者以逗号隔开的字符串指定。
tags | 类似分类 categories，一篇文章也可以给它增加一个或者多个标签。同样，tags 可以通过 YAML 列表或者以逗号隔开的字符串指定。

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
col1|col2|col3
-|-|-
a1|b1|c1
a2|b2|c2
a3|b3|c3
```

#### markdown 引用与链接

```
引用链接：
[显示文本](链接地址)

引用图片：
![](图片链接地址)

引用文字：
> 引用的文字

链接PDF：
[PDF名称](PDF链接地址)
```

#### markdown 高亮代码

```java
public static void main(String[] args) {
    System.out.println("Hello World!");
}
```


查阅 [Jekyll docs][jekyll-docs] 了解更多信息，Jekyll的Github的主页： [Jekyll’s Github repo][jekyll-gh] ，如有问题可访问 [Jekyll Talk][jekyll-talk] 进行讨论。

[jekyll-docs]: http://jekyllcn.com/docs/home/
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

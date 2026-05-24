---
title: 博客文章开头的 --- 到底是什么
published: 2026-05-24
description: 解释 YAML frontmatter 是什么、每个字段有什么用、哪些必填哪些可选
tags: [博客, Markdown, Astro]
category: 技术
draft: false
lang: zh_CN
---

如果你看过我博客的 Markdown 源文件，会发现每篇文章开头都有这样一段：

```markdown
---
title: 文章标题
published: 2026-05-24
description: 一段描述
tags: [标签1, 标签2]
category: 分类名
draft: false
lang: zh_CN
---
```

两个 `---` 之间的部分叫 **frontmatter**（直译就是"前面的内容"），用 YAML 格式写。它本质上是一段元数据——描述这篇文章"是什么"，而不是"写了什么"。

## 为什么要有 frontmatter

没有 frontmatter 的话，博客系统怎么知道这篇文章的标题是什么、什么时候发的、属于哪个分类？它当然可以自己去猜（比如把文件名叫当标题、用文件修改时间当发布日期），但那不可靠。

frontmatter 就是你自己把这些信息告诉系统的地方。Astro 在构建时会解析这段 YAML，然后把数据注入到页面模板中——标题显示在浏览器标签栏和文章列表里，日期用来排序，标签用来生成分类页面，等等。

## 字段说明

以下是这个博客（Fuwari + Astro）支持的所有字段，直接扒的类型定义。

### title（必填）

```yaml
title: 博客文章开头的 --- 到底是什么
```

文章的标题，会显示在浏览器标签栏、文章列表页、文章顶部。没有标题的文章在 RSS 里也没法看，所以这是唯一真正必填的字段。

### published（必填）

```yaml
published: 2026-05-24
```

发布日期，必须是 `YYYY-MM-DD` 格式。博客的文章列表按这个字段排序，不是按文件修改时间。

### updated（可选）

```yaml
updated: 2026-05-25
```

最后更新日期。如果你发完一篇文章后又大幅修改，加这个字段可以让读者知道文章不是过时的。不会替代 `published` 的排序作用。

### draft（可选，默认 false）

```yaml
draft: true
```

设为 `true` 时，构建会跳过这篇文章。适合写了一半不想发布的草稿。发布时删掉这行或者改成 `false` 就行。

### description（可选）

```yaml
description: 解释 YAML frontmatter 是什么、每个字段有什么用的、哪些必填哪些可选
```

文章摘要。会出现在文章列表页的卡片下方、RSS feed 里、搜索引擎结果中。不写的话列表页会截取正文前几个字代替，但不好控制截断位置，建议手动填。

### tags（可选）

```yaml
tags: [博客, Markdown, Astro]
```

标签列表。用于生成标签聚合页面，读者可以按标签浏览相关文章。不加也可以，但博客的标签页就看不到这篇文章了。

### category（可选）

```yaml
category: 技术
```

文章分类。跟 tags 的区别是：一篇文章通常只有一个分类但可以有多个标签。分类更像是目录，标签更像是关键词。比如这篇文章在"技术"分类下，但如果我写一篇游记，可能就在"生活"分类下。

这个博客的分类显示在文章列表的左上角小标签里。

### lang（可选）

```yaml
lang: zh_CN
```

文章语言代码。影响页面 `lang` 属性和日期格式。中文文章用 `zh_CN`，英文用 `en`。不写的话默认跟站点配置走。

### image（可选）

```yaml
image: /path/to/cover.png
```

文章封面图。会显示在文章列表卡片和文章顶部。我目前还没用过。

### prevTitle / prevSlug / nextTitle / nextSlug（内部使用）

```yaml
prevTitle: 上一篇
prevSlug: slug-of-previous-post
nextTitle: 下一篇
nextSlug: slug-of-next-post
```

这 4 个字段类型定义里标注了 "For internal use"，Fuwari 内部用来生成"上一篇/下一篇"导航链接的。如果你在文章列表页看到过文章底部有上一篇下一篇的链接，就是它们控制的。

实际上你不用自己写——Fuwari 会自动根据 `published` 日期排序来生成相邻文章的导航。只有在你想手动指定（比如写了一个系列，希望强制按顺序导航）时才需要填。

## 一个完整的例子

```markdown
---
title: 博客文章开头的 --- 到底是什么
published: 2026-05-24
updated: 2026-05-25
description: 解释 YAML frontmatter 是什么、每个字段有什么用
tags: [博客, Markdown, Astro]
category: 技术
draft: false
lang: zh_CN
image: /images/cover.png
---

正文从这里开始...
```

## 写的时候要注意什么

- **冒号后面必须有空格**（英文的 `title: xxx`，不是 `title:xxx`）
- **日期必须是 YYYY-MM-DD 格式**。写错了构建时会报错，因为类型定义写了 `z.date()`
- **tags 用方括号括起来，逗号分隔**。写错了 tags 页面可能找不到这篇文章
- **一行写一个字段**，不要把多个字段挤在一行里
- **正文从第二个 `---` 之后开始**，不要写在 frontmatter 里面

## 为什么要用 --- 分隔

用 `---` 作为分隔符是 YAML frontmatter 的事实标准，最早来自 Jekyll，后来被几乎所有的静态站点生成器继承：Hexo、Hugo、Astro、Vitepress……只要你看到 Markdown 文件开头有 `---`，就知道它是个静态博客或者文档站点。

这个博客用的 Fuwari 主题基于 Astro，而 Astro 的 frontmatter 处理继承了这一套规范——没有什么特别的，就是业界惯例。

---

以上就是在前面写 `---` 的全部意义。不是魔法，不是什么高级功能，就是告诉博客系统"这篇文章的基本信息在这里"而已。

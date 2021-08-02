---
title: 「Ringo」- 朴素的 Hexo 主题
tags:
  - Hexo
  - 博客
  - Ringo
categories: 技术
abbrlink: 4016581467
date: 2021-08-01 13:47:30
---

## 缘起

第一次接触 Hexo 是在一片教程上，当时采用的是教程中使用的 [Next](https://github.com/theme-next/hexo-theme-next) 主题。作为一款非常流行的 Hexo 主题，Next 确实十分优秀，外观简洁且功能完备，但在网络上还是太千篇一律。后来又改用了 [Material](https://github.com/bollnh/hexo-theme-material) 主题，但因为该主题长期未更新，换成了风格类似的 [Melody](https://github.com/Molunerfinn/hexo-theme-melody) 主题，过了一段时间又换成了基于 Melody 但功能更丰富的 [Butterfly](https://github.com/jerryc127/hexo-theme-butterfly) 主题。 ~~博客美化就和 Linux 桌面美化一样纠结。~~

## 为什么要自己写

> 当你看到你用的主题出现在两个以上的博客的时候，那你就要考虑自己写一个了。 ——《Hexo 主题开发指南》

这句话或许太过暴论，但确实说明了一点：博客主题应该是自己个人需要的主题，而不是直接从流行的教程上复制。虽然 [Butterfly](https://github.com/jerryc127/hexo-theme-butterfly) 主题功能丰富而且美观，但抱着写一个完全符合自己审美的主题以及锻炼前端水平的想法，最终仍然决定进行 [Ringo](https://github.com/HeliumOI/hexo-theme-ringo) 主题的开发。 ~~但说来说去还是在用别人的设计。~~

## Ringo

### 过程

Ringo 主题最初来自 [memset0](https://github.com/memset0) 开发的 Typecho 主题 [typecho-theme-ringo](https://github.com/memset0/typecho-theme-ringo)。但贫穷的 Sophon 并没有多余的零花钱能用来购买服务器，一直以来都在使用 Hexo 作为博客系统，所以就有了将其移植到 Hexo 上的想法。

因为在前端领域几乎没什么基础，所以开始时大多数地方都是在按照 landscape 等知名主题依样画葫芦，遇到问题则在 Google 上搜索大佬们的解决方案。其实 Hexo 的官方文档中对于一些 API 和机制都有比较详细的阐述，EasyHexo 则记录了一些小坑，很多问题都在 [Yifei Zhang](https://hqcfly.github.io/index.html) 大佬的 [Hexo 主题开发指南](https://hqcfly.github.io/2016/06/19/hexo-theme-guide/)中得到了解答。

实际开发遇到的问题主要有喜闻乐见的代码高亮等，阅读生成页面源码后发现 Hexo 实际上是使用 `table` 标签来支持行号显示，而主题的 CSS 中已经对表格进行了渲染，最终显示出来的代码便才不忍睹。因此最后的选择是关闭 Hexo 原生的 Highlight 代码渲染，在主题内部使用 Highlight。Ringo 主题支持的另一个代码高亮方案则是 Prettify，这是我所知道的几个代码渲染方案中唯一一个原生支持行号的，主要参考了 [SukkaW 大佬的博客](https://blog.skk.moe/post/add-prettify-to-hexo/)

一开始，主题中很多部分都使用了 jQuery，但在阅读了一些大佬们的博客后决定将使用 jQuery 的代码都换成原生 JS 实现，而负责图片显示的 FancyBox 因为本身就基于 jQuery，被换成了原生 JS 实现的 Viewer.js。由于经验不足，刚发布的主题中也有很多低级错误，例如在 `head` 标签中加载 JavaScript。

移植的过程中也因为静态博客和动态博客的区别遇到过一些问题。静态博客和动态博客的最大区别在于，动态博客可以直接调用博客系统提供的数据实现搜索等功能，而静态博客则不得不使用 JavaScript 在前端处理数据，这算是移植过程中遇到的困难之一。

博客的评论系统开始使用的是灵活的 Disqus，但考虑到通用性也增加了对 Gitalk 的支持，后来又加入了 Livere 和 Valine 评论系统。后来考虑到 Disqus 在国内访问不便，又使用了 [SukkaW](https://skk.moe) 大佬的 [DisqusJS](https://github.com/SukkaW/DisqusJS) 项目渲染评论列表。 ~~（因经济条件而使用 Cloudflare Workerers 搭建 DisqusJS 反向代理）~~

### 功能

一路踩坑后，还只是个雏形的 Ringo 主题已经拥有了以下功能：

- [x] 归档、标签、分类页面
- [x] i18n 支持
- [x] 访问次数统计
- [x] 数学公式（MathJax）
- [x] 代码高亮（Highlight.js / Prettify）
- [x] 图片（Viewer.js）
- [x] 无 jQuery，纯原生 JavaScript
- [x] 评论系统（Gitalk / Disqus / Valine / Livere）
- [x] 页脚备案信息
- [x] Google Analytics

## 关于主题

### 预览

- [Hexo-Theme-Ringo](https://ringo.sophonci117.me/)

![](https://cdn.jsdelivr.net/gh/HeliumOI/imghost@latest/ringo-demo.png)

### 下载

[HeliumOI/hexo-theme-ringo](https://github.com/HeliumOI/hexo-theme-ringo)

```bash
git clone https://github.com/HeliumOI/hexo-theme-ringo.git themes/ringo
```

求 Star qwq

### License

[![license](https://img.shields.io/github/license/HeliumOI/hexo-theme-ringo.svg?style=flat-square)](https://github.com/HeliumOI/hexo-theme-ringo/blob/master/LICENSE)

Open sourced under the GPL v3.0 license.

根据 GPL V3.0 许可证开源。
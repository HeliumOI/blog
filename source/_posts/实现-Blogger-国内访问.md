---
title: 实现 Blogger 国内访问
tags:
  - Blogger
  - 博客
  - Google
  - 域名
categories: 技术
abbrlink: 2811309648
date: 2021-08-03 11:13:23
---

## 关于 Blogger

[阮一峰](https://www.ruanyifeng.com/blog/)曾经提出过博客的“三个阶段”理论：

> 第一阶段，刚接触Blog，觉得很新鲜，试着选择一个免费空间来写。
> 第二阶段，发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。
> 第三阶段，觉得独立博客的管理太麻烦，最好在保留控制权的前提下，让别人来管，自己只负责写文章。

这一理论在现实中未必正确，因为在有个人服务器的情况下，更多的人还是愿意选择 WordPress 或 Typecho 等自建的动态博客。但在没有服务器时，按照阮一峰老师的说法使用 Hexo、Jekyll 等静态页面生成器并部署在 GitHub Pages 上同样不失为一个好的选择，但所需要的技术对部分人并不友好。

所以部分人更愿意在简书、CSDN 等博客平台上写作，由博客平台直接提供软件，自己只需要写作内容。这类博客平台的优点是便捷，SEO 好，缺点是界面单调，广告繁多，并且有时内容并不由自己控制。博客园是这类平台中较为优秀的一个，可以自定义网页样式，吸引了大量的用户，但前段时间的网站整改让很多人心寒。相较之下，Blogger 创立二十余年，作为世界上最早的博客平台之一，在被 Google 收购后功能更是不断晚上，各方面的指标都很优秀，但是作为 Google 的产品，Blogger 在国内访问有时并不稳定，这个问题可以靠自定义域名来解决。

## 自定义域名

点击 Blogger 的“设置 -> 正在发布 -> 自定义域名”，在其中输入你的域名。

然后再设置DNS记录，一下用 Cloudflare 演示：

![](https://cdn.jsdelivr.net/gh/HeliumOI/imghost@latest/blogger1.png)

（代理状态必须是“仅限DNS”，否则无法认证。）

将你的域名关联到 `ghs.google.com`。此时你的 Blogger 已经可以从自定义域名访问了，只是还不支持 HTTPS，所以还要在 Blogger 中将“设置 -> HTTPS -> HTTPS 可用性”打开，以产生你的自定义域名的 HTTPS 证书。这个操作需要十几分钟的时间，当“HTTPS 可用性”显示为“状态：可用”时就可以用 HTTPS 访问了。

![](https://cdn.jsdelivr.net/gh/HeliumOI/imghost@latest/blogger2.png)

## 替换资源

这时你的 Blogger 虽然已经能从国内访问了，但背景图片、头像、CSS、JavaScript 等资源仍然需要从 Google 的服务器加载，在国内仍然无法正常使用，所以我们需要编辑主题。

点击 “设置 -> 主题背景 -> 修改HTML”

### 背景图片

在主题源码中查找

```css
url(https://themes.googleusercontent.com/image?id=L1lcAxxz0CLgsDzixEprHJ2F38TyEjCyE3RSAjynQDks0lT1BDc1OxXKaTEdLc89HPvdB11X9FDw)
```

将内容替换成你自己的背景图片的地址。

### 头像

在主题源码中查找

```html
<img class='profile-img' expr:alt='data:messages.myPhoto' expr:height='data:authorPhoto.height' expr:src='data:authorPhoto.image' expr:width='data:authorPhoto.width'/>
```

替换为

```html
<img class='profile-img' src="你的头像地址">
```

### CSS、JavaScript

#### 屏蔽

将`</head>`替换为`&lt;/head&gt;&lt;!--</head>--&gt;`

将`</body>`替换为`&lt;!--</body>--&gt;&lt;/body&gt;`

这样一来，自动插入的 CSS 和 JavaScript 就会被包含在注释里，不会被浏览器加载。

#### 加载

`indie_compiled.js`这个文件必须被加载，否则页面无法正常显示，所以我们要把它替换成自己的资源。

在主题源码中查找

```html
<b:template-script async='true' name='indie' version='1.0.0'/>
```

将其替换为

```html
<script async='async' src='https://cdn.jsdelivr.net/gh/HeliumOI/imghost@latest/2404877392-indie_compiled.js'></script>
```

为了稳定，更好的办法是下载`https://cdn.jsdelivr.net/gh/HeliumOI/imghost@latest/2404877392-indie_compiled.js`这个文件，上传到自己的空间里再引用。

### 示例

示例可以参见[我的 Blogger](https://blog.sophonci117.me/)
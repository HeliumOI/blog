---
title: Hexo踩坑记录（一）
tags:
  - Hexo
  - 博客
  - Node.js
abbrlink: 2851103091
date: 2020-08-10 15:39:53
categories: 技术
---

最近几天翻找自己的学习笔记时，忽然发现自己的Hexo博客已经长了几个月的草，于是一时兴起又更新了一下Hexo博客，但在构建博客时出现了问题：

## Hexo与Node.js的兼容性


```
mike@MacBook-Pro ~/hexo $ hexo g                                            [15:13:57]
INFO  Validating config
INFO  Start processing
INFO  Files loaded in 3.05 s
(node:18613) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
(node:18613) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:18613) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:18613) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:18613) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:18613) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
INFO  Generated: tags/index.html
INFO  Generated: about/index.html
INFO  Generated: categories/index.html
INFO  Generated: archives/364215652.html
INFO  Generated: archives/2019/index.html
INFO  Generated: archives/2019/03/index.html
INFO  Generated: archives/2019/04/index.html
INFO  Generated: archives/2019/05/index.html
INFO  Generated: archives/2019/06/index.html
INFO  Generated: archives/2019/07/index.html
INFO  Generated: archives/2020/index.html
INFO  Generated: archives/2020/08/index.html
INFO  Generated: archives/2020/02/index.html
INFO  Generated: categories/游记/index.html
INFO  Generated: categories/网络/index.html
INFO  Generated: categories/随笔/index.html
INFO  Generated: categories/学习笔记/index.html
INFO  Generated: categories/题解/index.html
INFO  Generated: categories/洛谷/index.html
INFO  Generated: categories/洛谷/题解/index.html
INFO  Generated: categories/洛谷/题解/数学/index.html
INFO  Generated: tags/网络安全/index.html
INFO  Generated: tags/NOIP/index.html
INFO  Generated: tags/数据结构/index.html
INFO  Generated: tags/网络/index.html
INFO  Generated: tags/平衡树/index.html
INFO  Generated: tags/Telegram/index.html
INFO  Generated: tags/随笔/index.html
INFO  Generated: tags/转载/index.html
INFO  Generated: tags/天文/index.html
INFO  Generated: tags/SCP/index.html
INFO  Generated: tags/洛谷/index.html
INFO  Generated: tags/计算几何/index.html
INFO  Generated: tags/题解/index.html
INFO  Generated: img/algolia.svg
INFO  Generated: tags/SPOJ/index.html
INFO  Generated: archives/4021524966.html
INFO  Generated: archives/3790557036.html
INFO  Generated: archives/3811762271.html
INFO  Generated: archives/1389482586.html
INFO  Generated: archives/1301002429.html
INFO  Generated: archives/3299158642.html
INFO  Generated: archives/2032683215.html
INFO  Generated: archives/142292747.html
INFO  Generated: archives/1438989892.html
INFO  Generated: archives/2197869946.html
INFO  Generated: archives/3769932704.html
INFO  Generated: archives/3735749857.html
INFO  Generated: archives/index.html
INFO  Generated: archives/2552852216.html
INFO  Generated: archives/2078626702.html
INFO  Generated: img/icp.png
INFO  Generated: img/favicon.png
INFO  Generated: images/algolia_logo.svg
INFO  Generated: images/avatar.gif
INFO  Generated: images/apple-touch-icon-next.png
INFO  Generated: images/cc-by-nc-nd.svg
INFO  Generated: images/cc-by-nc-sa.svg
INFO  Generated: images/cc-by-nd.svg
INFO  Generated: images/cc-by.svg
INFO  Generated: images/cc-zero.svg
INFO  Generated: images/cc-by-nc.svg
INFO  Generated: images/cc-by-sa.svg
INFO  Generated: images/favicon-16x16-next.png
INFO  Generated: images/favicon.png
INFO  Generated: images/loading.gif
INFO  Generated: images/logo.svg
INFO  Generated: images/favicon-32x32-next.png
INFO  Generated: images/placeholder.gif
INFO  Generated: images/searchicon.png
INFO  Generated: images/quote-r.svg
INFO  Generated: images/quote-l.svg
INFO  Generated: archives/3581552066.html
INFO  Generated: archives/2251452003.html
INFO  Generated: archives/704167021.html
INFO  Generated: archives/1580237458.html
INFO  Generated: archives/1827780785.html
INFO  Generated: index.html
INFO  Generated: images/Jietu20190416-142149@2x.jpg
INFO  Generated: archives/2019/page/2/index.html
INFO  Generated: archives/page/2/index.html
INFO  Generated: img/404.jpg
INFO  Generated: css/var.css
INFO  Generated: archives/1401793044.html
INFO  Generated: page/2/index.html
INFO  Generated: img/loading.gif
INFO  Generated: page/3/index.html
INFO  Generated: archives/page/3/index.html
INFO  Generated: js/utils.js
INFO  Generated: js/search/local-search.js
INFO  Generated: js/third-party/ClickShowText.js
INFO  Generated: img/friend_404.gif
INFO  Generated: images/Jietu20190408-204825@2x.jpg
INFO  Generated: images/Jietu20190416-142524@2x.jpg
INFO  Generated: images/avatar2.jpg
INFO  Generated: js/search/algolia.js
INFO  Generated: js/third-party/activate-power-mode.js
INFO  Generated: js/third-party/canvas-nest.js
INFO  Generated: js/third-party/canvas-ribbon.js
INFO  Generated: js/third-party/click_heart.js
INFO  Generated: js/third-party/fireworks.js
INFO  Generated: js/third-party/piao.js
INFO  Generated: images/right_rotate.jpg
INFO  Generated: images/1583162215202.jpg
INFO  Generated: images/left_rotate.jpg
INFO  Generated: images/no.jpg
INFO  Generated: js/main.js
INFO  Generated: js/tw_cn.js
INFO  Generated: images/1582504823938.jpg
INFO  Generated: search.xml
INFO  Generated: css/index.css
INFO  Generated: images/avatar1.jpg
INFO  Generated: images/1583165079891.jpg
INFO  Generated: images/avatar.jpg
INFO  114 files generated in 1.74 s
```


首先，Sophon酱自然是发挥自己作为一名OIer搜（xun）索（zhao）资（ti）料（jie）的能力，找到了这个[Hexo的GitHub Issue](https://github.com/hexojs/hexo/issues/4257)

按照这里面的大牛的说法，这是Node.js 14的一个兼容性问题，Sophon酱使用的版本是这样的：

```
mike@MacBook-Pro ~/hexo $ hexo version                                      [15:14:07]
INFO  Validating config
hexo: 5.0.1
hexo-cli: 4.2.0
os: Darwin 19.6.0 darwin x64
node: 14.4.0
v8: 8.1.307.31-node.33
uv: 1.37.0
zlib: 1.2.11
brotli: 1.0.7
ares: 1.16.0
modules: 83
nghttp2: 1.41.0
napi: 6
llhttp: 2.0.4
openssl: 1.1.1g
cldr: 37.0
icu: 67.1
tz: 2019c
unicode: 13.0
```

Node.js 14目前与Hexo存在兼容性问题（目前Hexo官方让然没有修好），所以只要将Node.js降级就可以了。

以下是在macOS系统上的操作

```
brew unlink node #Homebrew操作，解除原来的Node.js在/usr/local/bin/内的链接
brew install node@12 #安装Node.js 12
brew link --overwrite --force node@12 #将新安装的Node.js 12连接到/usr/local/bin/node
```

## macOS升级后NPM更新报错

在写这篇文章之前，Sophon酱使用的Hexo版本号是4.2.1，在听说Hexo 5.0.1更新的消息后准备更新一下试试，然后就遇到了如下问题：

```
mike@MacBook-Pro ~/hexo $ npm install --save                               [13:48:33]

> fsevents@1.2.11 install /Users/mike/hexo/node_modules/nunjucks/node_modules/fsevents
> node-gyp rebuild

No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.

gyp: No Xcode or CLT version detected!
gyp ERR! configure error
gyp ERR! stack Error: `gyp` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onCpExit (/usr/local/Cellar/node@12/12.18.1/lib/node_modules/npm/node_modules/node-gyp/lib/configure.js:351:16)
gyp ERR! stack     at ChildProcess.emit (events.js:315:20)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:275:12)
gyp ERR! System Darwin 19.6.0
gyp ERR! command "/usr/local/Cellar/node@12/12.18.1/bin/node" "/usr/local/Cellar/node@12/12.18.1/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /Users/mike/hexo/node_modules/nunjucks/node_modules/fsevents
gyp ERR! node -v v12.18.1
gyp ERR! node-gyp -v v5.1.0
gyp ERR! not ok
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.11 (node_modules/nunjucks/node_modules/fsevents):
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.11 install: `node-gyp rebuild`
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: Exit status 1

updated 2 packages in 28.613s

9 packages are looking for funding
  run `npm fund` for details
```

其中，`
gyp: No Xcode or CLT version detected!`可以看出这个错误是Xcode工具造成的，因此更新一下Xcode工具：

```
mike@MacBook-Pro ~/hexo $ xcode-select --install                            [14:18:11]
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

emmmm好吧，Xcode工具并没有更新，那就只能是上个星期macOS升级时出现了问题。先删除原来的Xcode工具：

```
sudo rm -rf $(xcode-select -print-path)
```

再执行这条命令：

```
mike@MacBook-Pro ~/hexo $ xcode-select --install                            [14:19:06]
xcode-select: note: install requested for command line developer tools
```

这次再试试更新Hexo:

```
mike@MacBook-Pro ~/hexo $ npm install --save                                [14:19:19]

> fsevents@1.2.11 install /Users/mike/hexo/node_modules/nunjucks/node_modules/fsevents
> node-gyp rebuild

  SOLINK_MODULE(target) Release/.node
  CXX(target) Release/obj.target/fse/fsevents.o
  SOLINK_MODULE(target) Release/fse.node
added 71 packages from 33 contributors in 12.939s

10 packages are looking for funding
  run `npm fund` for details
```

大功告成！
---
title: Hexo更新到5.4.0
tags:
  - Hexo
  - 博客
  - Node.js
categories: 技术
abbrlink: 2192384774
date: 2021-07-28 10:58:46
---

## 缘起

因为更新后的的butterfly主题需要新版的Hexo，所以将Hexo更新到了5.4.0。但直接`npm install hexo`产生了一堆Warning，修改`package.json`有可能会破坏依赖。查找了相关资料后才得知更新Node.js项目需要使用`npm-check`和`npm-upgrade`两个包。

## 更新

### 安装

```bash
npm install -g npm-check
npm install -g npm-upgrade
```

### 使用

运行`npm-check`会检测可用的更新。

```bash
hexo                      😍  UPDATE!   Your local install is out of date. https://hexo.io/
                                       npm install --save hexo@5.4.0 to go from 5.0.1 to 5.4.0

hexo-abbrlink             😕  NOTUSED?  Still using hexo-abbrlink?
                                       Depcheck did not find code similar to require('hexo-abbrlink') or import from 'hexo-abbrlink'.
                                       Check your code before removing as depcheck isn't able to foresee all ways dependencies can be used.
                                       Use --skip-unused to skip this check.
                                       To remove this package: npm uninstall --save hexo-abbrlink

hexo-deployer-git         😎  MAJOR UP  Major update available. https://hexo.io/
                                       npm install --save hexo-deployer-git@3.0.0 to go from 2.1.0 to 3.0.0
                          😕  NOTUSED?  Still using hexo-deployer-git?
                                       Depcheck did not find code similar to require('hexo-deployer-git') or import from 'hexo-deployer-git'.
                                       Check your code before removing as depcheck isn't able to foresee all ways dependencies can be used.
                                       Use --skip-unused to skip this check.
                                       To remove this package: npm uninstall --save hexo-deployer-git
```

`npm-upgrade`则会更新你的`package.json`文件。

```bash
Checking for outdated dependencies for "/Users/sophon/Documents/blog/package.json"...
[====================] 15/15 100%

New versions of active modules available:

  hexo                    ^5.0.1   →   ^5.4.0
  hexo-deployer-git       ^2.1.0   →   ^3.0.0
  hexo-generator-search   ^2.4.0   →   ^2.4.3
  hexo-renderer-pug       ^1.0.0   →   ^2.0.0
  hexo-renderer-stylus    ^1.1.0   →   ^2.0.1

? Update "hexo" in package.json from ^5.0.1 to ^5.4.0? Yes

? Update "hexo-deployer-git" in package.json from ^2.1.0 to ^3.0.0? Yes

? Update "hexo-generator-search" in package.json from ^2.4.0 to ^2.4.3? Yes

? Update "hexo-renderer-pug" in package.json from ^1.0.0 to ^2.0.0? Yes

? Update "hexo-renderer-stylus" in package.json from ^1.1.0 to ^2.0.1? Yes


These packages will be updated:

  hexo                    ^5.0.1   →   ^5.4.0
  hexo-deployer-git       ^2.1.0   →   ^3.0.0
  hexo-generator-search   ^2.4.0   →   ^2.4.3
  hexo-renderer-pug       ^1.0.0   →   ^2.0.0
  hexo-renderer-stylus    ^1.1.0   →   ^2.0.1

? Update package.json? Yes
```

此时再运行`npm install --save`按照`package.json`文件内容进行更新。

查看Hexo版本：

```
hexo: 5.4.0
hexo-cli: 4.2.0
os: Darwin 20.5.0 darwin x64
node: 12.22.1
v8: 7.8.279.23-node.46
uv: 1.40.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.16.1
modules: 72
nghttp2: 1.41.0
napi: 8
llhttp: 2.1.3
http_parser: 2.9.4
openssl: 1.1.1k
cldr: 39.0
icu: 69.1
tz: 2021a
unicode: 13.0
```

完结，撒花！
---
title: 用 Cloudflare Workers 反代 Blogger
tags:
  - Blogger
  - 博客
  - Google
  - Cloudflare
  - SNI
categories: 技术
abbrlink: 1924021233
date: 2021-08-04 17:47:07
---

## 缘起

[上一篇文章](https://sophonci117.me/archives/2811309648/)中提到了利用自定义域名搭建可以在国内访问的 Blogger 博客的方法，但经过讨论和测试，这个方法存在以下几个问题：

1. 文章中的图片等资源会被 Google 服务器自动缓存。对国外的用户来说，这一机制可以大大提高页面加载的速度，但对国内用户来说则恰恰相反。目前除了在 Markdown 编辑器的预览中直接复制富文本外还没有别的解决方案。

2. 地址 ghs.google.com 并不一定能解析到一个国内可以访问的地址。虽然只要使用自定义域名就可以规避 DNS 污染和 SNI 阻断，但是 Google 的服务器受到的是特殊待遇，直接针对 IP 地址进行阻断，自定义域名也经常抽风。

反观另一个同样无法访问的网站 Pixiv，网上给出的解决方案是直接在本地运行 Nginx 进行反代就可以访问。这个解决方案本质上是利用了 SNI 协议的漏洞，即为了区分前往同一地址的不同域名，SNI 协议中会以明文展示你需要前往的域名。明文展示显然会导致在特定的地区访问不稳定等种种问题，所以互联网领域的开发者们创造了 ESNI，后来又再 ESNI 的基础上创造了 ECHO，然后是 ECH……然而根本问题是大多数网站根本就不打算去支持 ECH。为了解决在特定地区访问不稳定的问题，给 SNI 加密固然是一个方法，但另一个更简单粗暴的方法是：弄一个假的 SNI 或者根本不发 SNI！

（这里插播一个小故事：运营商有时会给官网等网站免流量，就是用 SNI 来判断你访问的是那个网站。曾经有人利用这个机制盗刷了价值数十万元的流量，然后成功地被判刑了……）

总而言之，网络上给出的 Pixiv 反代方案其实就是利用了 Nginx 反代时不支持 SNI，并且 Pixiv 并没有直接被封锁 IP，这就是为什么同样的方案不能被用来访问 Google，Blogger 使用自定义域名也偶尔抽风。

对于问题 2，套一层 CDN 看似是一个解决方案，但实践表明这会导致包括但不限于循环重定向/HTTPS 证书错误/404 Not Found 等问题……

## Cloudflare Workers

### 缘起

[@PetrichorArk](https://t.me/PetrichorArk) 提出可以用 CF Workers 反代来解决上述问题。简单来说就是用一个 CF Workers 在访问时爬取 Blogger 的页面，将页面中的资源进行替换后再反会给访问者。~~（万能的 Cloudflare Workers）~~

### 代码实现

以下是 [@PetrichorArk](https://t.me/PetrichorArk) 编写的实现代码：

```javascript
/**
 * URL:
 * https://something.something.workers.dev/
 */

addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})

const blogHost = 'something.blogspot.com'

/**
 * @param {Map} error
 * @param {Number} status
 * @param {Boolean} cacheable
 */
function handleInvalidRequest(error, status, cacheable) {
    const response = new Response(JSON.stringify(error), {
        status: status,
        statusText: 'Invalid Request',
        headers: {
            'strict-transport-security': 'max-age=31536000; includeSubDomains; preload',
            'timing-allow-origin': '*',
            'x-server': 'blog-proxy-2cff9aba',
            'x-xss-protection': '1; mode=block'
        }
    })
    if (cacheable) {
        response.headers.set('cache-control', 'public, max-age=29030400, immutable')
    }
    return response
}

/**
 * @param {String} url
 */
async function fromCache(url) {
    const cache = caches.default
    const matched = await cache.match(url)
    if (matched) {
        return matched
    }
    const resp = await fetch(url)
    if (resp.status >= 200 && resp.status < 300) {
        const response = new Response(resp.body, {
            status: resp.status,
            statusText: resp.statusText,
            headers: resp.headers
        })
        response.headers.delete('expires')
        response.headers.delete('vary')
        response.headers.delete('access-control-allow-origin')
        response.headers.set('cache-control', 'public, max-age=29030400, immutable')
        response.headers.set('strict-transport-security', 'max-age=31536000; includeSubDomains; preload')
        response.headers.set('timing-allow-origin', '*')
        response.headers.set('x-mirrored-url', url)
        response.headers.set('x-server', 'blog-proxy-2cff9aba')
        response.headers.set('x-xss-protection', '1; mode=block')
        await cache.put(url, response.clone())
        return response
    }
    return handleInvalidRequest({
        msg: 'status_error',
        url: url,
    }, resp.status, false)
}

/**
 * @param {URL} url
 */
async function proxy(url) {
    const proxyHost = url.hostname
    url.hostname = blogHost
    const urlStr = url.href
    const resp = await fetch(urlStr)
    if (resp.status >= 200 && resp.status < 400) {
        let body
        const type = resp.headers.get('content-type')
        if (type && type.startsWith('text/')) {
            body = await resp.text()
            body = body.replaceAll(blogHost, proxyHost)
            body = body.replace(new RegExp(`<link href='(.*?)${proxyHost}/(.*?)' rel='canonical'/>`), `<link href='$1${blogHost}/$2' rel='canonical'/>`)
            body = body.replace(/lh\w*?.googleusercontent.com/g, proxyHost + '/_image')
        } else {
            body = resp.body
        }
        const response = new Response(body, {
            status: resp.status,
            statusText: resp.statusText,
            headers: resp.headers
        })
        response.headers.delete('vary')
        response.headers.delete('access-control-allow-origin')
        response.headers.set('strict-transport-security', 'max-age=31536000; includeSubDomains; preload')
        response.headers.set('timing-allow-origin', '*')
        response.headers.set('x-mirrored-url', urlStr)
        response.headers.set('x-server', 'blog-proxy-2cff9aba')
        response.headers.set('x-xss-protection', '1; mode=block')
        return response
    }
    return handleInvalidRequest({
        msg: 'status_error',
        url: urlStr,
    }, resp.status, false)
}

/**
 * @param {Request} request
 */
async function handleRequest(request) {
    let url
    try {
        url = new URL(request.url)
    } catch {
        return handleInvalidRequest({ msg: 'url_parse_error', url: request.url }, 400, true)
    }
    if (url.pathname.startsWith('/_image/')) {
        url.hostname = 'lh3.googleusercontent.com'
        url.pathname = url.pathname.substr(7)
        return await fromCache(url)
    }
    return await proxy(url)
}
```

使用时记得将代码开头处的 `blogHost` 赋值为你自己的 Blogger 的 Blogspot 域名。原先的域名要接触与 Blogger 的绑定，这样 CF Workers 才能访问到你的真实博客页面而不是一个重定向页面。再将你的自定义域名解析到你的 CF Workers。（并在 Workers 面板设置路由。）

## 总结

Blogger 的地址 `ghs.google.com` 如果解析到国内可以访问的 IP，访问速度其实比 Cloudflare 的节点更快，但对于主题模版的修改的资源的处理则让人头大。相比之下，Cloudflare Workers 反代的方式更稳定也更便捷。~~不过折腾得这么麻烦为什么不直接用 Hexo，果然生命不止折腾不息吗。~~
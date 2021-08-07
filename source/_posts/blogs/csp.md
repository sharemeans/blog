---
title: 关于浏览器的CSP（Content-Security-Policy）
categories: web安全
tags: [CSP]
date: 2021-8-3
---

在浏览器命令行执行eval函数时，发现在有些网页的devTools中执行时报错：

```
VM132:1 Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src github.githubassets.com".

    at new Function (<anonymous>)
    at <anonymous>:1:15
(anonymous) @ VM132:1
```
而在其它大部分网页的devTools中执行是正常的。

根据报错信息大致明白：该网站设置了`Content Security Policy`，且该安全策略不允许执行`unsafe-eval`类型的脚本。

拿github的为例，其html的response header有这个字段`content-security-policy`，其值格式化之后如下：

```
default-src 'none';
base-uri 'self';
block-all-mixed-content;
connect-src 'self' uploads.github.com www.githubstatus.com collector.githubapp.com api.github.com github-cloud.s3.amazonaws.com github-production-repository-file-5c1aeb.s3.amazonaws.com github-production-upload-manifest-file-7fdce7.s3.amazonaws.com github-production-user-asset-6210df.s3.amazonaws.com cdn.optimizely.com logx.optimizely.com/v1/events translator.github.com wss://alive.github.com github.githubassets.com;
font-src github.githubassets.com;
form-action 'self' github.com gist.github.com;
frame-ancestors 'none';
frame-src render.githubusercontent.com render-temp.githubusercontent.com viewscreen.githubusercontent.com;
img-src 'self' data: github.githubassets.com identicons.github.com collector.githubapp.com github-cloud.s3.amazonaws.com secured-user-images.githubusercontent.com/ *.githubusercontent.com customer-stories-feed.github.com spotlights-feed.github.com;
manifest-src 'self';
media-src github.githubassets.com;
script-src github.githubassets.com;
style-src 'unsafe-inline' github.githubassets.com;
worker-src github.com/socket-worker-3f088aa2.js gist.github.com/socket-worker-3f088aa2.js
```
关于这些字段的意义参考文末的资料。大致明白浏览器的安全策略主要涉及到script、img、font、xhr、style、媒体文件等资源的来源域名，防止页面被第三方恶意插入不信任域名的代码。

我们的项目目前都是裸奔，至于这些设置能达到哪些程度上的安全，由于目前对web安全研究甚少，这块完全是盲点，打算认真啃下《Web前端黑客技术揭秘》这本书。

### 参考资料
https://www.ruanyifeng.com/blog/2016/09/csp.html

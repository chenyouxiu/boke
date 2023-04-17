---
layout: '../../layouts/MarkdownPost.astro'
title: 'Shell变成第一天'
pubDate: 2035-06-01
description: ''
author: 'Han'
cover:
    url: 'https://pic.lookcos.cn/i/usr/uploads/2022/04/2067928922.png'
    square: 'https://pic.lookcos.cn/i/usr/uploads/2022/04/2067928922.png'
    alt: 'cover'
tags: ["源码研究", "标准库", "golang", "gin"]
theme: 'light'
featured: false
---

![Go HTTP Server的大致处理流程|wide](https://pic.lookcos.cn/i/usr/uploads/2023/02/3697706570.png)

服务器在收到请求时，首先进入路由 Router，接着路由会根据 request 请求的路径，找到对应的处理器(Handler)，处理器再根据 request 进行处理并构造 response 进行返回。

## 利用标准库实现一个简单HTTP Server

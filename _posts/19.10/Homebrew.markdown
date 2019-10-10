---
layout:     post
title:      "Homebrew切换源"
subtitle:   " \"Homebrew change source\""
date:       2019-10-10 20:00:00
author:     "Dsyzxml"
header-img: "img/in-post/PostBackground/ab.jpeg"
catalog: true
tags:
    - Homebrew
    - Git
---

> "Homebrew是Mac上的一个包管理工具。"

Homebrew是基于Git的，本身也是Github上的一个开源项目。
在国内由于墙的原因，使用Homebrew时往往会碰到各种问题。有两种解决这个问题的办法。最简单的当然是使用Shadowsocks等翻墙工具啦。但是由于需要租服务器，没钱的小伙伴要怎么办呢？
其实，国内也有许多Homebrew的镜像：

> "清华的镜像:[https://mirror.tuna.tsinghua.edu.cn/help/homebrew/](https://mirror.tuna.tsinghua.edu.cn/help/homebrew/)"

> "阿里的镜像:[https://mirrors.aliyun.com/homebrew/brew.git](https://mirrors.aliyun.com/homebrew/brew.git)"

**更换brew.git**

```
cd "$(brew --repo)"

git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
```

**更换homebrew-core.git**

```
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"

git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
```


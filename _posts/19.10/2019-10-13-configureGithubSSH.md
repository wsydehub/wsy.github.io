---
layout:     post
title:      "配置Github的ssh"
subtitle:   ""
date:       2019-10-13 18:00:00
author:     "Dsyzxml"
header-img: "img/in-post/PostBackground/ab.jpeg"
catalog: true
tags:
    - Git
    - ssh
---

## 本地使用ssh生成一对公钥和私钥(如果已有就不用再生成了)

```
ssh-keygen -t rsa -C "youremali@xx.xx"
```
如果你没有改变默认路径,生成的密钥会放在***~/.ssh***下

## 将生成的公钥加入到你的Github的钥匙串
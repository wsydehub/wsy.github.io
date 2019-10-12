---
layout:     post
title:      "Python的classmethod和staticmethod"
subtitle:   " \"The classmethod and staticmethod of Python\""
date:       2019-10-12 17:00:00
author:     "Dsyzxml"
header-img: "img/in-post/PostBackground/ab.jpeg"
catalog: true
tags:
    - Python
---

在尝试用Python编写PortScanner是遇到了@classmethod的注释，学习一下(没有系统学过Python，勿喷^*^)
>一个例子

```
class A(object):
    def __init__(self, id):
        self.id = id

    def normal(self):
        return self

    @staticmethod
    def staticM(id):
        return id

    @classmethod
    def classM(self):
        return self

if __name__ == '__main__':
    instanceA = A(1)

    print("NormalMethod: {}\n".format(instanceA.normal))

    print("ClassMethodCalledByClass: {0}\nClassMethodCalledByInstance: {1}\n".
          format(A.classM, instanceA.classM))

    print("StaticMethod: {}".format(A.staticM))

```

> 结果

```
NormalMethod: <bound method A.normal of <__main__.A object at 0x1109e7210>>

ClassMethodCalledByClass: <bound method A.classM of <class '__main__.A'>>
ClassMethodCalledByInstance: <bound method A.classM of <class '__main__.A'>>

StaticMethod: <function A.staticM at 0x1109c3b90>
```

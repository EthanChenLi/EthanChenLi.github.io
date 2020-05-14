---
layout: post
title:  "Golang设计模式-单例模式"
date:   2020-5-14 17:20:00 +0800
categories: go
tags: go
comments: 1
---

##### 定义
123
##### 需要注意
456
##### 代码示例

```
    package main

    import (
        "sync"
    )
    var _instance *single
    var once sync.Once

    type single struct {}

    //创建单例模式
    func GetInstance() *single{
        //once 它能保证某个操作仅且只执行一次
        once.Do(func() {
            _instance = &single{}
        })
        return _instance
    }
 ```
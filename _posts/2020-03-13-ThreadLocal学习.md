---
layout: post
title: ThreadLocal 学习
categories: [Java]
description: 
keywords: 
---

* content
{:toc}


## 简介

ThreadLocal 类提供了线程局部 (thread-local) 变量。这些变量与普通变量不同，每个线程都可以通过其 get 或 set方法来**访问自己的独立初始化的变量副本**。ThreadLocal 实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。

## 整体认识

![http://www.milky.show/images/java/threadlocal/th_1.png](http://www.milky.show/images/java/threadlocal/th_1.png)


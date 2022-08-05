---
layout: post
title:  "3 Wayland book Libwayland in depth"
date:   2022-08-02 21:13:29 +0800
categories: jekyll update
---
# The libwayland implementation

We spoke briefly about libwayland in chapter 1.3 &mdash; the most popular 
Wayland implementation. Much of this book is applicable to any implementation,
but we're going to spend the next two chapters familiarizing you with this one.

在 1.3 小节中我们曾简单介绍过 libwayland 库，它是目前 Wayland 协议实现中最流行的一种。
本书大部分内容适用于任何一种 Wayland 实现库，但在接下来的两个章节里，我们将带你一起深入了解一下 libwayland。

The Wayland package includes pkg-config specs for wayland-client and
wayland-server &mdash; consult your build system's documentation for 
instructions on linking with them. Naturally, most applications will only link
to one or the other. The library includes a few simple primitives (such as a
linked list) and a pre-compiled version of `wayland.xml` &mdash; the core
Wayland protocol.

Wayland 软件包包含用于 wayland-client 和 wayland-server 的 pkg-config spec &mdash; 
你可以通过查询系统构建文档,获取到链接这两个库文件的指导说明。当然，对于大多数应用程序而言，
它们只会链接到其中一个（一般 client 链接的是 libwayland-client.so 而 server 则是 libwayland-server.so）。
该库包括一些简单的基本组成单元（如链接列表），以及一个 `wayland.xml` 的预编译版本，也就是
Wayland 协议的核心实现。

We'll start by introducing the primitives.

我们将从对基本单元（primitive）的介绍开始。

---
layout: post
title:  "3.1 Wayland book Libwayland in depth:Wayland-util primitives"
date:   2022-08-02 21:16:29 +0800
categories: jekyll update
---
# wayland-util primitives

Common to both the client and server libraries is `wayland-util.h`, which
defines a number of structs, utility functions, and macros that establish a
handful of primitives for use in Wayland applications. Among these are:

`wayland-util.h` 是 client 和 server 库（即 `libwayland-client.so` 和 `libwayland-server.so`）
都会访问到的公共头文件。它定义了许多结构体、辅助函数和宏定义，这些（定义）为构建
wayland 应用程序提供了一组十分有用的基本单元。它包括：

- Structures for marshalling & unmarshalling Wayland protocol messages in
  generated code
- A linked list (`wl_list`) implementation
- An array (`wl_array`) implementation (rigged up to the
  corresponding Wayland primitive)
- Utilities for conversion between Wayland scalars (such as fixed-point
  numbers) and C types
- Debug logging facilities to bubble up information from libwayland internals

- Wayland 协议消息打包（发送方）和解包（接收方）的统一格式
- 一个链表（`wl_list`）的具体实现
- 一个数组（`wl_array`）的具体实现（可变数组，用于存储相关的 wayland 基本单元）
- 辅助函数，用于转换 C 数据类型（如 int 整型/float 浮点型）和 wayland 数据类型（如 `wl_fixed_t`，定点数类型）
- 调试记录工具，用于从 libwayland 内部提取协议信息

The header contains many comments which document itself. The documentation is
quite good &mdash; you should read through the header yourself. We'll go into 
detail on how to apply these primitives in the next several pages.

该头文件包含许多优秀的注释，你完全可以自行阅读。在接下来的几页中我们将详细介绍
如何运用这些基本单元。

译者注：解释下什么叫 wayland 的 primitives，wayland 协议定义了一套 `client/server` 
相互解析消息的规则，这些规则由许多约定俗成的基本对象构筑而成，比如 display 代表一个
显示连接、surface 代表一个窗口、buffer 代表一个要显示的内容、interface 代表一个接口类型，
listener 代表一组用于监听消息的回调函数，当我们对这些基本对象有了足够的了解后，我们就会
对 wayland 协议有比较深刻的理解。


原文链接:https://wayland-book.com/

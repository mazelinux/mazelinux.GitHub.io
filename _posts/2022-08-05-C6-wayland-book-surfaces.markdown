---
layout: post
title:  "6 Wayland book Buffers & surfaces"
date:   2022-08-05 113:39:29 +0800
categories: jekyll update
---
# Buffers & surfaces

Apparently, the whole point of this system is to display information to users
and receive their feedback for additional processing. In this chapter, we'll
explore the first of these tasks: showing pixels on the screen.

显然，该系统的全部意义在于向用户显示信息并接收他们的反馈以进行额外处理。在本章中，我们将探索第一个任务：在屏幕上显示像素。

There are two primitives which are used for this purpose: buffers and surfaces,
governed respectively by the `wl_buffer` and `wl_surface` interfaces. Buffers
act as an opaque container for some underlying pixel storage, and are supplied
by clients with a number of methods &mdash; shared memory buffers and GPU 
handles being the most common.

buffer 和 surface 两个基本单元用于此目的，分别由 `wl_buffer` 和 `wl_surface` 接口管理。缓冲区充当一些底层像素存储的不透明容器，
由客户端提供多种方法（实现） &mdash; 共享内存缓冲区和 GPU handle 是最常见的。

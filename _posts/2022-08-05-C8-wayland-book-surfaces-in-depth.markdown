---
layout: post
title:  "8 Wayland book Surfaces in depth"
date:   2022-08-05 14:52:29 +0800
categories: jekyll update
---
# Surfaces in depth

The basic areas of the surface interface that we've shown until now are
sufficient to present data to the user, but the surface interface offers many
additional requests and events for more efficient use. Many &mdash; if not most 
&mdash; applications do not need to redraw the entire surface each frame. Even 
deciding *when* to draw the next frame is best done with the assistance of the
compositor. In this chapter, we'll explore the features of `wl_surface` in
depth.

其实到目前此，我们讲解的关于 surface 的接口的基本作用域（或者说这些接口本身的能力），
已经能够将（pixel）数据展示给用户，但是 surface 也提供了一些额外的 request 和 event
接口以便于更有效的工作。比方说，许多 &mdash;（不是大多数情况 &mdash;）应用程序并不需要
每一帧都进行整个表面的绘画。更进一步的，(应用程序) *何时* 绘画下一帧最好也是在 compositor
的帮助下决定。本章，我们将深入探讨 `wl_surface` 的特性。

原文链接:https://wayland-book.com/

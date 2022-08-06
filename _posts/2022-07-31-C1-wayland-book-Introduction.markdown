---
layout: post
title:  "1 Wayland-book Introduction"
date:   2022-07-31 22:10:29 +0800
categories: waylandbook
---
# Introduction

Wayland is the next-generation display server for Unix-like systems, designed
and built by the alumni of the venerable Xorg server, and is the best way to get
your application windows onto your user's screens. Readers who have worked with
X11 in the past will be pleasantly surprised by Wayland's improvements, and
those who are new to graphics on Unix will find it a flexible and powerful
system for building graphical applications and desktops.

Wayland 是面向 Unix-like 系统的下一代 Display-Server，是由备受尊敬的 Xorg-Server 的
设计者们所构建开发的，是将应用程序窗口放置到用户屏幕上的最佳选择。曾经使用过 X11 的
读者，在面对 Wayland 时，将会对 Wayland （对比于 X11）的改进感到惊喜，而那些对 Unix
图形系统不熟悉的人，将会发现，它在构建图形相关的应用程序和桌面方面，是多么的灵活而强
大。

This book will help you establish a firm understanding of the concepts, design,
and implementation of Wayland, and equip you with the tools to build your own
Wayland client and server applications. Over the course of your reading, we'll
build a mental model of Wayland and establish the rationale that went into its
design. Within these pages you should find many "aha!" moments as the intuitive
design choices of Wayland become clear, which should help to keep the pages
turning. Welcome to the future of open source graphics!

本书将帮助你对 Wayland 整体概念、设计方法和具体实现有一个深入的理解，并为你提供构建自
己的 Wayland 客户端和服务器程序的工具。在阅读过程中，我们将在大脑里，构建出一个 Wayland
的大致模型，并建立对其设计的基本原理的认知。在接下来的内容中，随着对 Wayland 设计（方法）
理解进一步清晰直观，你会发现许多 "aha" 的时刻，这应该有助于保持页面的滚动[你能继续看下去]。
欢迎来到未来开源图形（设计）！

**Notice**: this is a *draft*. Chapters 1-10 are more or less complete, but may
be updated later. Chapters 11 forward in large part remain to be written.

**注意**：这本书只是一个 *草稿*。第 1-10 章节或多或少的已经完成，未来可能会更新。而第
11 章往后的大部分内容还有待完成。

**TODO**

- Expand on resource lifetimes and avoiding race conditions in chapter 2.4
- Move linux-dmabuf details to the appendix, add note about wl_drm & Mesa
- Rewrite the introduction text
- Add example code for interactive move, to demonstrate the use of serials
- Prepare PDFs and EPUBs


- 在第 2.4 章中扩展关于资源生命周期和避免竞争条件（的内容）
- 将 linux-dmabuf 详细信息移至附录，添加有关 wl_drm 和 Mesa 的注释
- 重写 introduction 章节内容
- 添加 interactive move 章节的示例代码，以演示 serials 的使用
- 准备 PDF 和 EPUB

## About the book

This work is licensed under a <a rel="license"
href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons
Attribution-ShareAlike 4.0 International License</a>. The source code is
[available here][source].

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>

[source]: https://git.sr.ht/~sircmpwn/wayland-book

## About the author

<small>
  In the words of Preston Carpenter, a close collaborator of Drew's:
</small>

<small>
  用 Drew 的亲密合作伙伴 Preston Carpenter 的话来说：
</small>

Drew DeVault got his start in the Wayland world by building sway, a clone of the
popular tiling window manager i3. It is now the most popular tiling Wayland
compositor by any measure: users, commit count, and influence. Following its
success, Drew gave back to the Wayland community by starting wlroots:
unopinionated, composable modules for building a Wayland compositor. Today it is
the foundation for dozens of independent compositors, and Drew is one of the
foremost experts in Wayland.

Drew DeVault 最开始接触 Wayland 世界是因为构建 sway ，（sway）是流行的平铺窗口管理器
i3 的克隆版。不管以何种标准（来排序），它都是现在最流行的平铺方式的 Wayland compositor：
用户量、（社区）提交计数和影响力。继其成功之后，Drew 回到 Wayland 社区，开始 wlroots 项目：
用于构建 Wayland compositor 的无限制的、可组合的模块。如今是数十个相对独立的 compositor
的基础，Drew 是 Wayland 最重要的专家之一。

原文链接:https://wayland-book.com/

---
layout: post
title:  "10 Wayland book XDG shell in depth"
date:   2022-08-05 15:12:29 +0800
categories: jekyll update
---
# XDG shell in depth

So far we've managed to display something on-screen in a top-level application
window, but there's more to XDG shell that we haven't fully appreciated yet.
Even the simplest application would be well-served to implement the
configuration lifecycle correctly, and xdg-shell offers useful features to more
complex application as well. The full breadth of xdg-shell's feature set
includes client/server negotiation on window size, multi-window hierarchies,
client-side decorations, and semantic positioning for windows like context
menus.

到目前为止，我们已经成功地在顶级应用程序窗口中在屏幕上显示了一些东西，但是 XDG shell
还有更多我们还没有完全理解的东西。 即使是最简单的应用程序也可以很好地正确实现配置生命周期，
而 xdg-shell 也为更复杂的应用程序提供了有用的功能。 xdg-shell 的全部功能集包括客户端/服务器
协商窗口大小、多窗口层次结构、客户端装饰和上下文菜单等窗口的语义定位。

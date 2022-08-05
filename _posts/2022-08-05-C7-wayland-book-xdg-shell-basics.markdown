---
layout: post
title:  "7 Wayland book XDG shell basics"
date:   2022-08-05 13:50:29 +0800
categories: jekyll update
---
# XDG shell basics

The XDG (cross-desktop group) shell is a standard protocol extension for Wayland
which describes the semantics for application windows. It defines two
`wl_surface` roles: "toplevel", for your top-level application windows, and
"popup", for things like context menus, dropdown menus, tooltips, and so on -
which are children of top-level windows. With these together, you can form a
tree of surfaces, with a toplevel at the top and popups or additional toplevels
at the leaves. The protocol also defines a *positioner* interface, which is used
for help positioning popups with limited information about the things around
your window.

XDG（Cross-Desktop Group，跨桌面平台小组）shell 是基于 Wayland 标准协议的一个扩展(协议)，
它描述了应用程序窗口的基本操作语法。它定义了两种 `wl_surface` 角色：toplevel，用于顶层应
用程序窗口；popup，用于上下文菜单、下拉菜单、工具提示框等：它们是顶层窗口的子窗口（或子集）。
将这些结合在一起，你可以形成一个关于 surface 的树型结构，其顶部有一个 toplevel，而叶分支上
则是 popup 或其它 toplevel。该扩展协议还定义了一个 *positioner* 接口，通过该窗口周围的局部
信息，来帮助定位 popup 类型的 wl_surface。

xdg-shell, as a protocol *extension*, is not defined in `wayland.xml`. Instead
you'll find it in the `wayland-protocols` package. It's probably installed at a
path somewhat like `/usr/share/wayland-protocols/stable/xdg-shell/xdg-shell.xml`
on your system.

xdg-shell 作为 Wayland 协议的一个扩展，并没有在 `wayland.xml` 中定义，而是定义在了 `wayland-protocols`
软件包中。通常，它被安装到形如 `/usr/share/wayland-protocols/stable/xdg-shell/xdg-shell.xml`
的路径下。

## xdg_wm_base

`xdg_wm_base` is the only global defined by the specification, and it provides
requests for creating each of the other objects you need. The most basic
implementation starts by handling the "ping" event - when the compositor sends
it, you should respond with a "pong" request in a timely manner to hint that you
haven't become deadlocked. Another request deals with the creation of
positioners, the objects mentioned earlier, and we'll save the details on these
for chapter 10. The request we want to look into first is `get_xdg_surface`.

`xdg_wm_base` 是该扩展定义的唯一全局 object，当你想要创建基于该扩展的其他 object 时，
可以通过绑定该全局 object 来发起 request。最基本的实现从处理 “ping” event 开始：当 compositor
发送 “ping” event 时，你（client 端）应该及时响应 “pong” request 以表明你处于可响应状态
（译者注：否则的话，就类似微软的 Windows 操作系统，有时候遇见的“窗口未响应”）。另一个 request 则用于创建我们前面提到的
positioner （定位器）对象，我们将在第 10 章对其展开讨论。这里，我们首先来研究一下 `get_xdg_surface` request。

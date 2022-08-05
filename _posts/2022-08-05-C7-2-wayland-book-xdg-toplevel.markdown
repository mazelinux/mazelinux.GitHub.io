---
layout: post
title:  "7.2 Wayland book Application windows"
date:   2022-08-05 13:55:29 +0800
categories: jekyll update
---
# Application windows

We have shaved many yaks to get here, but it's time: XDG toplevel is the
interface which we will finally use to display an application window. The XDG
toplevel interface has many requests and events for managing application
windows, including dealing with minimized and maximized states, setting window
titles, and so on. We'll be discussing each part of it in detail in future
chapters, so let's just concern ourselves with the basics now.

历经千辛万苦，终于到达了这儿，是时候了：我们将用 XDG toplevel 接口显示应用程序
窗口。XDG toplevel 接口有许多用于管理应用程序窗口的 request 和 event，包括最小
化和最大化状态处理、设置窗口标题等等。我们将在以后的章节中详细讨论它的每个实现，
但是现在，我们只关注基本知识。

Based on our knowledge from the last chapter, we know that we can obtain an
`xdg_surface` from a `wl_surface`, but that it only constitutes the first step:
bringing a surface into the fold of XDG shell. The next step is to turn that XDG
surface into an XDG toplevel &mdash; a "top-level" application window, so named
for its top-level position in the hierarchy of windows and popup menus we will
eventually create with XDG shell. To create one of these, we can use the
appropriate request from the `xdg_surface` interface:

根据上一章的知识，我们知道我们可以从 `wl_surface` 中获得一个 `xdg_surface`，这只
是第一步：将一个 surface 带入 XDG shell 上下文中。下一步是将该 XDG surface 转换为
XDG toplevel：一个 “toplevel” 应用程序窗口。之所以这样命名，是因为它在基于 XDG shell
创建的窗口和弹出菜单的层次结构中是处于顶层位置的。要创建 toplevel，我们可以使用 `xdg_surface`
的 request 接口：

```xml
<request name="get_toplevel">
  <arg name="id" type="new_id" interface="xdg_toplevel"/>
</request>
```

This new `xdg_toplevel` interface puts many requests and events at our disposal
for managing the lifecycle of application windows. Chapter 10 explores these in
depth, but I know you're itching to get something on-screen. If you follow these
steps, handling the `configure` and `ack_configure` riggings for XDG surface
discussed in the previous chapter, and attach and commit a `wl_buffer` to our
`wl_surface`, an application window will appear and present your buffer's
contents to the user. Example code which does just this is provided in the next
chapter. It also leverages one additional XDG toplevel request which we haven't
covered yet:

新接口 `xdg_toplevel` 将提供许多 request 和 event，用于管控应用程序窗口的生命周期。
第 10 章将深入探讨这些，不过我知道你现有多么渴望想在屏幕上显示些东西出来。按照如下
步骤操作：处理上一章中讨论的 XDG surface 的 `configure` 和 `ack_configure`，将 `wl_buffer`
attach 到我们的 `wl_surface`，最终完成 commit（提交）动作，（如果一切顺利的话）将会
显示一个带有 buffer 内容的应用程序窗口。执行此操作的示例代码将在下一章中提供。此外，
它还利用了我们尚未涉及的一个额外的 XDG toplevel 的 request：

```xml
<request name="set_title">
  <arg name="title" type="string"/>
</request>
```

This should be fairly self-explanatory. There's a similar one that we don't use
in the example code, but which may be appropriate for your application:

该 request 看名字就知道什么意思了，这里就不解释了。我们没有在示例代码中使用类似的方法，
但它可能适用于你的应用程序：

```xml
<request name="set_app_id">
  <arg name="app_id" type="string"/>
</request>
```

The title is often shown in window decorations, tasksbars, etc, whereas the app
ID is used to identify your application or group your windows together. You
might utilize these by setting your window title to "Application windows &mdash;
The Wayland Protocol &mdash; Firefox", and your app ID to "firefox".

title 通常显示在窗口装饰、任务栏等中，而 app ID 用于标识你的应用程序，或者将你的窗口分组。
你可以通过这些接口将窗口标题设置为 “Application windows — The Wayland Protocol — Firefox”，
并将你的应用程序 ID 设置为 “firefox”。

In summary, the following steps will take you from zero to a window on-screen:

简要概括，如下步骤将带领你完成从零开始显示一个窗口到屏幕上的全过程：

1. Bind to `wl_compositor` and use it to create a `wl_surface`.
1. Bind to `xdg_wm_base` and use it to create an `xdg_surface` with your
   `wl_surface`.
1. Create an `xdg_toplevel` from the `xdg_surface` with
   `xdg_surface.get_toplevel`.
1. Configure a listener for the `xdg_surface` and await the `configure` event.
1. Bind to the buffer allocation mechanism of your choosing (such as `wl_shm`)
   and allocate a shared buffer, then render your content to it.
1. Use `wl_surface.attach` to attach the `wl_buffer` to the `wl_surface`.
1. Use `xdg_surface.ack_configure`, passing it the serial from `configure`,
   acknowledging that you have prepared a suitable frame.
1. Send a `wl_surface.commit` request.

1. 绑定 `wl_compositor`，创建 `wl_surface`。
2. 绑定 `xdg_wm_base` ，通过 `wl_surface` 创建 `xdg_surface`。
3. 使用 `xdg_surface.get_toplevel` 接口，通过 `xdg_surface` 创建 `xdg_toplevel`。
4. 为 `xdg_surface` 配置一个 listener 并监听 `configure` event。
5. 绑定到你选择的 buffer 分配机制（例如 `wl_shm`），分配一个共享 buffer，然后将你的
内容放到 buffer 上。
6. 使用 `wl_surface.attach` 将 `wl_buffer` 绑定到 `wl_surface`。
7. 使用 `xdg_surface.ack_configure`，将 `configure` 中的序列号传递给它，确认你已经
准备了合适的一帧。
8. 发送 `wl_surface.commit` request。

Turn the page to see these steps in action.

点击下一页查看这些步骤的实际用法。

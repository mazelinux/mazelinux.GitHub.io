---
layout: post
title:  "7.1 Wayland book XDG surfaces"
date:   2022-08-05 13:52:29 +0800
categories: jekyll update
---
# XDG surfaces

Surfaces in the domain of xdg-shell are referred to as `xdg_surfaces`, and this
interface brings with it a small amount of functionality common to both kinds of
XDG surfaces &mdash; toplevels and popups. The semantics for each kind of XDG
surface are different enough still that they must be specified explicitly
through an additional role.

我们使用 `xdg_surfaces` 来表示 xdg-shell 协议下的 surface，尽管该接口为两种不同角色
（toplevel 和 popup）的 XDG surface 提供了小部分通用功能。但是，由于不同 XDG surface
的用法差异较大，因此还必须使用额外的角色（toplevel 或 popup）来明确它们。

The `xdg_surface` interface provides additional requests for assigning the more
specific roles of popup and toplevel. Once we've bound an object to the
`xdg_wm_base` global, we can use the `get_xdg_surface` request to obtain one for
a `wl_surface`.

`xdg_surface` 接口提供了一些额外的 request，用来给 xdg_surface 分配更加具体的角色（popup 
或 toplevel）。一旦我们将一个 object 绑定到 `xdg_wm_base` 这个全局 object 上，我们就
可以使用 `get_xdg_surface` 这个 request，为 `wl_surface` 获取一个 xdg_surface。

```xml
<request name="get_xdg_surface">
  <arg name="id" type="new_id" interface="xdg_surface"/>
  <arg name="surface" type="object" interface="wl_surface"/>
</request>
```

The `xdg_surface` interface, in addition to requests for assigning a more
specific role of toplevel or popup to your surface, also includes some important
functionality common to both roles. Let's review these before we move on to the
toplevel- and popup-specific semantics.

（如前面所述）`xdg_surface` 接口，除了可以为你的 surface 请求分配更具体的 toplevel
或 popup 角色外，还针对两种角色共有的部分，提供了许多重要功能。在继续讨论 toplevel
和 popup 的具体用法之前，让我们先了解一下这些共有的功能。

```xml
<event name="configure">
  <arg name="serial" type="uint" summary="serial of the configure event"/>
</event>

<request name="ack_configure">
  <arg name="serial" type="uint" summary="the serial from the configure event"/>
</request>
```

The most important API of xdg-surface is this pair: `configure` and
`ack_configure`. You may recall that a goal of Wayland is to make every frame
perfect. That means no frames are shown with a half-applied state change, and to
accomplish this we have to synchronize these changes between the client and
server. For XDG surfaces, this pair of messages is the mechanism which supports
this.

xdg-surface 最重要的 API 就是这一对：`configure` 和 `ack_configure`。 你可能还记得，
Wayland 的一大目标是让每一帧都完美无缺。也就是说，它绝对不会将一个只更新了一半的画面
（frame）显示出来，为了实现这一点，我们必须在 client 和 server 之间同步这些修改。对于
XDG surface，这对消息机制正是用来实现这一点的。

We're only covering the basics for now, so we'll summarize the importance of
these two events as such: as events from the server inform your configuration
(or reconfiguration) of a surface, apply them to a pending state. When a
`configure` event arrives, apply the pending changes, use `ack_configure` to
acknowledge you've done so, and render and commit a new frame. We'll show this
in practice in the next chapter, and explain it in detail in chapter 8.1.

由于本章只介绍基础知识，因此这里我们简单总结一下这两个事件的重要性：当来自 server
的 event 通知你对 surface 进行配置（或重新配置）时，先将它们（配置的具体内容）置于 pending
状态。只有等到 `configure`  event 到达时，才让之前处于 pending 状态的修改彻底生效。
并使用 `ack_configure` 确认您已完成此操作，紧接着渲染并提交新的一帧画面。我们将在
下一节的实践操练中演示该过程，并在第 8.1 章中详细解释。

```xml
<request name="set_window_geometry">
  <arg name="x" type="int"/>
  <arg name="y" type="int"/>
  <arg name="width" type="int"/>
  <arg name="height" type="int"/>
</request>
```

The `set_window_geometry` request is used primarily for applications using
client-side decorations, to distinguish the parts of their surface which are
considered a part of the window, and the parts which are not. Most commonly,
this is used to exclude client-side drop-shadows rendered behind the window from
being considered a part of it. The compositor may apply this information to
govern its own behaviors for arranging and interacting with the window.

`set_window_geometry` request 主要用于那些使用了 client 装饰的应用程序，用来区分一个
surface 中，哪些区域是窗口的一部分，哪些区域不是。最常见的用法，则是使用该请求来排除窗口
后面呈现的阴影区域（client-side drop-shadow）。compositor可以通过该信息来管理窗口排列
和窗口交互的具体行为。

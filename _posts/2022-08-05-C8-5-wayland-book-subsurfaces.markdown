---
layout: post
title:  "8.5 Wayland book Subsurfaces"
date:   2022-08-05 14:58:29 +0800
categories: waylandbook
---
# Subsurfaces

There's only one[^1] surface role defined in the core Wayland protocol,
`wayland.xml`: subsurfaces. They have an X, Y position relative to the parent
surface &mdash; which needn't be constrained by the bounds of their parent 
surface &mdash; and a Z-order relative to their siblings and parent surface.

在核心 Wayland 协议中只定义了一个[^1]表面角色，`wayland.xml`：subsurfaces。 它们具有
相对于父 surface 的 X、Y 位置（不需要受其父 surface 的边界约束）以及相对于其兄弟 surface
和父 surface 的 Z-order。

Some use-cases for this feature include playing a video surface in its native
pixel format with an RGBA user-interface or subtitles shown on top, using an
OpenGL surface for your primary application interface and using subsurfaces to
render window decorations in software, or moving around parts of the UI without
having to redraw on the client. With the assistance of hardware planes, the
compositor, too, might not even have to redraw anything for updates your
subsurfaces. On embedded systems in particular, this can be especially useful
when it fits your use-case. A cleverly designed application can take advantage
of subsurfaces to be very efficient.

一些基于此功能的用例包括，以原始像素格式播放视频表面[一般来说数据是 yuv]，顶部显示
RGBA 格式的用户界面或字幕，使用 OpenGL 绘画表面作为你的主要应用程序界面，并使用子
表面在应用程序中渲染窗口相关部分，或围绕移动 UI 的部分，而不必在整个客户端上重绘。
在硬件平面(overlay)的帮助下，合成器甚至可能不需要重新绘制任何内容来更新你的子表面。
特别是在嵌入式系统上，在适合你的用例时特别有用。一个设计巧妙的应用程序可以利用子表
面来提高效率。

The interface for managing these is the `wl_subcompositor` interface. The
`get_subsurface` request is the main entry-point to the subcompositor:

`wl_subcompositor` 管理这些的接口。`get_subsurface` 请求是 subcompositor 的主入口：

```xml
<request name="get_subsurface">
  <arg name="id" type="new_id" interface="wl_subsurface" />
  <arg name="surface" type="object" interface="wl_surface" />
  <arg name="parent" type="object" interface="wl_surface" />
</request>
```

Once you have a `wl_subsurface` object associated with a `wl_surface`, it
becomes a child of that surface. Subsurfaces can themselves have subsurfaces,
resulting in an ordered tree of surfaces beneath any top-level surface.
Manipulating these children is done through the `wl_subsurface` interface:

一旦拥有与 `wl_surface` 关联的 `wl_subsurface` 对象，它(wl_subsurface)就会成为该
表面(wl_surface)的子级。子表面本身也可以具有子表面，从而在任何顶级表面下方生成一个
有序的表面树（状结构）。操作这些子表面是通过 `wl_subsurface` 接口完成的：

```xml
<request name="set_position">
  <arg name="x" type="int" summary="x coordinate in the parent surface"/>
  <arg name="y" type="int" summary="y coordinate in the parent surface"/>
</request>

<request name="place_above">
  <arg name="sibling" type="object" interface="wl_surface" />
</request>

<request name="place_below">
  <arg name="sibling" type="object" interface="wl_surface" />
</request>

<request name="set_sync" />
<request name="set_desync" />
```

A subsurface's z-order may be changed by placing it above or below any sibling
surface that shares the same parent, or the parent surface itself.

子表面可以通过将其放置在共享同一父表面的同级表面之上或之下，或者父表面本身的之上或之下
来更改它的z-order。

The synchronization of the various properties of a `wl_subsurface` requires some
explanation. These position and z-order properties are synchronized with the
parent surface's lifecycle. When a `wl_surface.commit` request is sent for the
main surface, all of its subsurfaces have changes to their position and z-order
applied with it.

`wl_subsurface` 的各种属性的同步需要详细说明一下。默认情况下，这些位置和 z-order 属性
与父表面的生命周期同步。当为主表面发送 wl_surface.commit 请求时，其所有子表面的位置和
z-order 都会发生变化。

However, the `wl_surface` state associated with this subsurface, such as the
attachment of buffers and accumulation of damage, need not be linked to the
parent surface's lifecycle. This is the purpose of the `set_sync` and
`set_desync` requests. Subsurfaces synced with their parent surface will commit
all of their state when the parent surface is committed. Desynced surfaces will
manage their own commit lifecycle like any other.

然而，与该子表面相关的 `wl_surface` 状态，例如缓冲区的附着(attach)和损伤(damage)，不需要
与父表面的生命周期相关联。这就是 `set_sync` 和 `set_desync` 请求的目的。 当父表面被提交时，
与其父表面同步的子表面将提交它们的所有状态。不同步的表面将像任何其他表面一样管理自己的提交
生命周期。

In short, the sync and desync requests are non-buffered and apply immediately.
The position and z-order requests are buffered, and are not affected by the
sync/desync property of the surface &mdash; they are always committed with the 
parent surface. The remaining surface state, on the associated `wl_surface`, is
committed in accordance with the sync/desync status of the subsurface.

简而言之，同步和非同步请求是非缓存机制的，会立即应用。position 和 z-order 请求被缓存，
并且不受表面的同步/非同步属性的影响——它们总是与父表面一起提交。关联在的 `wl_surface` 上
的剩余子表面状态根据子表面自身的同步/非同步状态决定提交策略。

[^1]: Disregarding the deprecated `wl_shell` interface.
[^1]：`wl_shell` 接口,被弃用的。

原文链接:https://wayland-book.com/

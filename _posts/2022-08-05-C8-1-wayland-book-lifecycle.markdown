---
layout: post
title:  "8.1 Wayland book Surface lifecycle"
date:   2022-08-05 13:54:29 +0800
categories: jekyll update
---
# Surface lifecycle

We mentioned earlier that Wayland is designed to update everything atomically,
such that no frame is ever presented in an invalid or intermediate state. Of the
many attributes that can be configured for an application window and other
surfaces, the driving mechanism behind that atomicity is the `wl_surface`
itself.

我们之前提过，Wayland 旨在以原子性的方式更新所有内容，因此不会出现任何一帧处于无效
或中间状态。在为应用程序窗口和其他 surface 配置种种属性的过程中，背后的原子性是基于
`wl_surface` 驱动的。

Every surface has a *pending* state and an *applied* state, and no state at all
when it's first created. The *pending* state is negotiated over the course of
any number of requests from clients and events from the server, and when both
sides agree that it represents a consistent surface state, the surface is
*committed* &mdash; and the pending state is *applied* to the current state of 
the surface. Until this time, the compositor will continue to render the last
consistent state; once committed, will use the new state from the next frame
forward.

除了第一次创建时没有状态，每个 surface 对象都有一个 *pending* 状态和一个 *applied* 状态。
*挂起* 状态作用于客户端的任意数量的请求和服务端对应的事件的协商（交互）过程，当双方一致
同意它（pending state）所代表的 surface 状态时，surface被 *提交*——并且挂起状态被应用于
当前表面状态。直到这个时候，合成器才会继续基于之前协商一致的状态进行渲染相关操作；（应用程序）
一旦提交完成，（合成器）将在下一帧显示给用户之前使用新状态。

Among the state which is updated atomically are:

需要原子更新的状态包括：

- The attached `wl_buffer`, or the pixels making up the content of the surface
- The region which was "damaged" since the last frame, and needs to be redrawn
- The region which accepts input events
- The region considered opaque[^1]
- Transformations on the attached `wl_buffer`, to rotate or present a subset of
  the buffer
- The scale factor of the buffer, used for HiDPI displays

- 附加的 `wl_buffer`，或构成表面内容的像素
- 基于上一帧来说的 "损坏" 的区域，也就是需要重新绘制的区域
- 能够接受输入事件的区域
- 被认为不透明的区域[^1]
- 对附加的 `wl_buffer` 进行转换，以旋转或裁剪的方式呈现缓冲区
- 缓冲区的缩放因子，用于 HiDPI 显示器

In addition to these features of the surface, the surface's *role* may have
additional double-buffered state like this. All of this state, along with any
state associated with the role, is applied when `wl_surface.commit` is sent. You
can send these requests several times if you change your mind, and only the most
recent value for any of these properties will be considered when the surface is
eventually committed.

表面不仅有这些特征，表面的 *角色* 可能还有像这样的额外的双缓冲状态。在发送 wl_surface.commit 时，
surface 会应用所有这些状态以及与角色关联的任何状态。如果你需要修改这些状态，可以多次发送这些请求，
在最终提交 surface 时，只会考虑这些属性设置中的最新值。

When you first create your surface, the initial state is invalid. To make it
valid (or to *map* the surface), you need to provide the necessary information
to build the first consistent state for that surface. This includes giving it a
role (like `xdg_toplevel`), allocating and attaching a buffer, and configuring
any role-specific state for that surface. When you send a `wl_surface.commit`
with this state correctly configured, the surface becomes valid (or *mapped*)
and will be presented by the compositor.

首次创建 surface 时，初始状态无效。为了使其有效（或 *映射* 表面），你需要提供必要的
信息来为该表面构建第一个一致状态。 这包括为其赋予角色（如 xdg_toplevel）、分配和附加缓冲区，
以及为该表面配置任何特定于角色的状态。 当你正确配置状态，并发送 wl_surface.commit 时，
表面变为有效（或 *映射*）并将由合成器呈现。

The next question is: when should I prepare a new frame?

下一个问题是：我应该什么时候准备一个新的帧？

[^1]: This is used by the compositor for optimization purposes.

[^1]：用于合成器优化（合成逻辑）。

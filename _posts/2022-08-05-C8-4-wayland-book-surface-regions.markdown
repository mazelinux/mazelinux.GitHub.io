---
layout: post
title:  "8.4 Wayland book Surface regions"
date:   2022-08-05 14:57:29 +0800
categories: waylandbook
---
# Surface regions

We've already used the `wl_compositor` interface to create `wl_surfaces` via
`wl_compositor.create_surface`. Note, however, that it has a second request:
`create_region`.

我们已经使用 `wl_compositor` 接口的 `wl_compositor.create_surface` 创建了 `wl_surfaces`。
但是请注意，它还有第二个请求接口：`create_region`。

```xml
<interface name="wl_compositor" version="4">
  <request name="create_surface">
    <arg name="id" type="new_id" interface="wl_surface" />
  </request>

  <request name="create_region">
    <arg name="id" type="new_id" interface="wl_region" />
  </request>
</interface>
```

The `wl_region` interface defines a group of rectangles, which collectively make
up an arbitrarily shaped region of geometry. Its requests allow you to do
bitwise operations against the geometry it defines by adding or subtracting
rectangles from it.

`wl_region` 接口定义了一组矩形，它们共同构成了一个任意形状的几何区域。它的请求接口
允许你通过添加或减去矩形来对其定义的几何图形进行操作。

```xml
<interface name="wl_region" version="1">
  <request name="destroy" type="destructor" />

  <request name="add">
    <arg name="x" type="int" />
    <arg name="y" type="int" />
    <arg name="width" type="int" />
    <arg name="height" type="int" />
  </request>

  <request name="subtract">
    <arg name="x" type="int" />
    <arg name="y" type="int" />
    <arg name="width" type="int" />
    <arg name="height" type="int" />
  </request>
</interface>
```

To make, for example, a rectangle with a hole in it, you could:

例如，要制作一个有中空的矩形，你可以：

1. Send `wl_compositor.create_region` to allocate a `wl_region` object.
2. Send `wl_region.add(0, 0, 512, 512)` to create a 512x512 rectangle.
3. Send `wl_region.subtract(128, 128, 256, 256)` to remove a 256x256 rectangle
   from the middle of the region.
   
1. 发送 `wl_compositor.create_region` 来分配一个 `wl_region` 对象。
2. 发送 `wl_region.add(0, 0, 512, 512)` 来创建一个 512x512 的矩形。
3. 发送 `wl_region.subtract(128, 128, 256, 256)` 以从区域中间移除一个 256x256 的矩形。

These areas can be disjoint as well; it needn't be a single continuous polygon.
Once you've created one of these regions, you can pass it into the `wl_surface`
interface, namely with the `set_opaque_region` and `set_input_region` requests.

这些区域也可以是不相交的；它不必是单个连续多边形。创建任意一个区域后，你可以将
其传递到 wl_surface 接口，即使用 set_opaque_region 和 set_input_region 请求。

```xml
<interface name="wl_surface" version="4">
  <request name="set_opaque_region">
    <arg name="region" type="object" interface="wl_region" allow-null="true" />
  </request>

  <request name="set_input_region">
    <arg name="region" type="object" interface="wl_region" allow-null="true" />
  </request>
</interface>
```

The opaque region is a hint to the compositor as to which parts of your surface
are considered opaque. Based on this information, they can optimize their
rendering process. For example, if your surface is completely opaque and
occludes another window beneath it, then the compositor won't waste any time on
redrawing the window beneath yours. By default, this is empty, which assumes
that any part of your surface might be transparent. This makes the default case
the least efficient but the most correct.

不透明区域对合成器来说是一个提示，即你的表面的哪些部分被认为是不透明的。基于这些信息，
他们（合成器）可以优化他们的渲染过程。例如，如果你的表面完全不透明并遮挡了它下方的另
一个窗口，那么合成器将不会浪费任何时间重新绘制你下方的窗口。 默认情况下，不透明区域是
空的，它假设表面的任何部分都可能是透明的。这种情况下效率最低，但最正确。

The input region indicates which parts of your surface accept pointer and touch
input events. You might, for example, draw a drop-shadow underneath your
surface, but input events which happen in this region should be passed to the
client beneath you. Or, if your window is an unusual shape, you could create an
input region in that shape. For most surface types by default, your entire
surface accepts input.

输入区域指示表面的哪些部分接受鼠标和触摸输入事件。例如，你可能会在你的表面下方绘制一个
阴影区域，但是在该区域发生的输入事件应该传递给你下方的客户端。或者，如果你的窗口是一个
不寻常的形状，你可以创建一个该形状的输入区域。默认情况下，对于大多数表面类型，整个表面
都接受输入事件。

Both of these requests can be used to set an empty region by passing in null
instead of a `wl_region` object. They're also both double-buffered &mdash; so 
send a `wl_surface.commit` to make your changes effective. You can destroy the
`wl_region` object to free up its resources as soon as you've sent the
`set_opaque_region` or `set_input_region` requests with it. Updating the region
after you send these requests will not update the state of the surface.

这两个请求都可用于通过传入 null 而不是 `wl_region` 对象来设置空区域。 它们也是双缓冲的——因此
发送 `wl_surface.commit` 以使你的更改生效。 你可以在发送 `set_opaque_region` 或
`set_input_region` 请求后立即销毁 `wl_region` 对象以释放其资源。发送这些请求后更新区域
不会更新表面的状态。

原文链接:https://wayland-book.com/

---
layout: post
title:  "8.6 Wayland book High density surfaces (HiDPI)"
date:   2022-08-05 14:59:29 +0800
categories: jekyll update
---
# High density surfaces (HiDPI)

In the past several years, a huge leap in pixel density in high-end displays has
been seen, new displays packing twice as many pixels into the same physical area
as we've seen in years past. We call these displays "HiDPI", short for "high
dots per inch". However, these displays are so far ahead of their "LoDPI" peers
that application-level changes are necessary to utilize them properly. By
doubling the screen resolution in the same space, we would halve the size of all
of our user interfaces if we lent them no special consideration. For most
displays, this would make the text unreadable and the interactive elements
uncomfortably small.

在过去几年中，高端显示器的像素密度出现了巨大飞跃，新显示器在同一物理区域中的像素数量
是我们过去几年看到的两倍。我们称这些显示器为 “HiDPI”，是“每英寸高点数”的缩写。然而，
这些显示器远远领先于它们的 “LoDPI” 同行，需要应用程序级别的更改才能正确利用它们。 通
过在同一空间内将屏幕分辨率加倍，如果我们不给它们特殊考虑，我们会将所有用户界面的大小减半。
对于大多数显示器，这会使文本不可读并且交互元素小得令人不舒服。

In exchange, however, we're offered a much greater amount of graphical fidelity
with our vector graphics, most notably with respect to text rendering. Wayland
addresses this by adding a "scale factor" to each output, and clients are
expected to apply this scale factor to their interfaces. Additionally, clients
which are unaware of HiDPI signal this limitation through inaction, allowing the
compositor to make up for it by scaling up their buffers. The compositor signals
the scale factor for each output via the appropriate event:

然而，作为交换，我们为矢量图形提供了更高的图形保真度，尤其是在文本渲染方面。 Wayland
通过为每个输出添加一个“比例因子”来解决这个问题，并且客户应该将这个比例因子应用到他们的接
口上。 此外，不知道 HiDPI 的客户端会通过不作为来表示此限制，从而允许合成器通过扩展其缓冲
区来弥补这一限制。 合成器通过适当的事件通知每个输出的比例因子：

```
<interface name="wl_output" version="3">
  <!-- ... -->
  <event name="scale" since="2">
    <arg name="factor" type="int" />
  </event>
</interface>
```

Note that this was added in version 2, so when binding to the `wl_output` global
you must set the version to at least 2 to receive these events. This is *not*
enough to decide to use HiDPI in your clients, however. In order to make that
call, the compositor must also send `enter` events for your `wl_surface` to
indicate that it has "entered" (is being shown on) a particular output or
outputs:

请注意，这是在版本 2 中添加的，因此在绑定到 wl_output 全局时，您必须将版本设置为至少 2
才能接收这些事件。 然而，这还不足以决定在您的客户端中使用 HiDPI。 为了进行该调用，合成器还
必须为您的 wl_surface 发送输入事件，以指示它已“输入”（正在显示）一个或多个特定输出：

```
<interface name="wl_surface" version="4">
  <!-- ... -->
  <event name="enter">
    <arg name="output" type="object" interface="wl_output" />
  </event>
</interface>
```

Once you know the collection of outputs a client is shown on, it should take the
maximum value of the scale factors, multiply the size (in pixels) of its buffers
by this value, then render the UI at 2x or 3x (or Nx) scale. Then, indicate the
scale the buffer was prepared at like so:

一旦您知道显示客户端的输出集合，它应该采用比例因子的最大值，将其缓冲区的大小（以像素为单位）
乘以该值，然后以 2x 或 3x（或 Nx）呈现 UI 规模。 然后，指出缓冲区的准备规模，如下所示：

```
<interface name="wl_surface" version="4">
  <!-- ... -->
  <request name="set_buffer_scale" since="3">
    <arg name="scale" type="int" />
  </request>
</interface>
```

**Note**: this requires version 3 or newer of `wl_surface`. This is the version
number you should pass to the `wl_registry` when you bind to `wl_compositor`.

注意：这需要 wl_surface 版本 3 或更新版本。 这是绑定到 wl_compositor 时应该传递给 wl_registry
的版本号。

Upon the next `wl_surface.commit`, your surface will assume this scale factor.
If it's greater than the scale factor of an output the surface is shown on, the
compositor will scale it down. If it's less than the scale factor of an output,
the compositor will scale it up.

在下一次 wl_surface.commit 时，您的表面将采用此比例因子。 如果它大于显示表面的输出的比例因子，
合成器会将其缩小。 如果它小于输出的比例因子，合成器将放大它。


原文链接:https://wayland-book.com/

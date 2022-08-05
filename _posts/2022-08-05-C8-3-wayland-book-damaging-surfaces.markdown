---
layout: post
title:  "8.3 Wayland book Damaging surfaces"
date:   2022-08-05 13:56:29 +0800
categories: jekyll update
---
# Damaging surfaces

You may have noticed in the last example that we added this line of code when we
committed a new frame for the surface:

你也许已经注意到，上个示例中，在提交新帧的时候我们调用了如下代码：

```
wl_surface_damage_buffer(state->wl_surface, 0, 0, INT32_MAX, INT32_MAX);
```

If so, sharp eye! This code *damages* our surface, indicating to the compositor
that it needs to be redrawn. Here we damage the entire surface (and well beyond
it), but we could instead only damage part of it.

如果是的话，说明你很敏锐！这段代码 *damage* 了我们的表面，向合成器表明它（这部分区域）
需要重新绘制。在这里，我们损坏了整个表面（以及远远超出它表面的范围），但实际上我们只能
损坏它（整个output窗口）的一部分。

Let's say, for example, that you've written a GUI toolkit and the user is typing
into a textbox. That textbox probably only takes up a small part of the window,
and each new character takes up a smaller part still. When the user presses a
key, you could render just the new character appended to the text they're
writing, then damage only that part of the surface. The compositor can then copy
just a fraction of your surface, which can speed things up considerably -
especially for embedded devices. As you blink the caret between characters,
you'll want to submit damage for its updates, and when the user changes views,
you'll likely damage the entire surface. This way, everyone does less work, and
the user will thank you for their improved battery life.

例如，假设你编写了一个 GUI 工具包，并且用户正在输入文本框。该文本框可能只占窗口的一小部分，
而每个新字符占更小部分。当用户按下一个键时，你可以只渲染他们正在编写的文本的新字符，然后只
将表面的那部分标记为损坏。然后合成器可以只复制表面的一小部分，这可以大大加快速度 - 特别是对
于嵌入式设备。当你在字符之间插入符号时，你将希望将其标记为损坏，为其更新提交，并且当用户更改
视图时，你可能会将整个表面标记为损坏。这样一来，每个人的工作量都会减少，用户会感谢你们改善了
他们电池的寿命。

**Note**: The Wayland protocol provides two requests for damaging surfaces:
`damage` and `damage_buffer`. The former is effectively deprecated, and you
should only use the latter. The difference between them is that `damage` takes
into account all of the transforms affecting the surface, such as rotations,
scale factor, and buffer position and clipping. The latter instead applies
damage relative to the buffer, which is generally easier to reason about.

**注意**：Wayland 协议提供了两个损坏表面的请求：`damage` 和 `damage_buffer`。前者
实际上已被弃用，你应该只使用后者。它们之间的区别在于损坏考虑了影响表面的所有变换，
例如旋转、缩放比例因子、缓冲区位置变化和裁剪。后者施加相对于缓冲区的损坏，这通常更
容易理解。

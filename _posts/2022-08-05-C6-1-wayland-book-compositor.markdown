---
layout: post
title:  "6.1 Wayland book Using wl_compositor"
date:   2022-08-05 13:42:29 +0800
categories: jekyll update
---
# Using wl_compositor

They say naming things is one of the most difficult problems in computer
science, and here we are, with evidence in hand. The `wl_compositor` global is
the Wayland compositor's, er, compositor. Through this interface, you may
send the server your windows for presentation, to be *composited* with the
other windows being shown alongside it. The compositor has two jobs: the
creation of surfaces and regions.

人们常说，命名是计算机科学中最困难的问题之一，就像现在这样。`wl_compositor` global对象是
这个 Wayland compositor 的 compositor。通过这个接口，你可以把你的窗口发给 server，以便（server）
将它和其他窗口混合，显示（在屏幕上）。合成器有两项工作：创建 surface 和创建 region。


To quote the spec, a Wayland *surface* has a rectangular area which may be
displayed on zero or more outputs, present buffers, receive user input, and
define a local coordinate system. We'll take all of these apart in detail later,
but let's start with the basics: obtaining a surface and attaching a buffer to
it. To obtain a surface, we first bind to the `wl_compositor` global. By
extending the example from chapter 5.1 we get the following:

如 Spec 定义，一个 Wayland *surface* 有一个矩形的区域，可以在零个或者多个 output 上显示，承载 buffer，
接收用户的输入，或者定义一个局部的坐标。稍后我们会详细介绍这些细节，目前先让我们从最基础的开始：获取一个
surface 并且为其附加 buffer。为了获取一个 surface，我们首先需要绑定 global 对象 `wl_compositor`。如下，
扩展第 5.1章中的实列：

```
struct our_state {
    // ...
    struct wl_compositor *compositor;
    // ...
};

static void
registry_handle_global(void *data, struct wl_registry *wl_registry,
		uint32_t name, const char *interface, uint32_t version)
{
    struct our_state *state = data;
    if (strcmp(interface, wl_compositor_interface.name) == 0) {
        state->compositor = wl_registry_bind(
            wl_registry, name, &wl_compositor_interface, 4);
    }
}

int
main(int argc, char *argv[])
{
    struct our_state state = { 0 };
    // ...
    wl_registry_add_listener(registry, &registry_listener, &state);
    // ...
}
```

Note that we've specified version 4 when calling `wl_registry_bind`, which is
the latest version at the time of writing. With this reference secured, we can
create a `wl_surface`:

请注意我们在调用 `wl_registry_bind` 时指定了版本 4，（此版本）是撰写本文时的最新版本。
在正确使用调用函数后，将创建一个 `wl_surface`。

```
struct wl_surface *surface = wl_compositor_create_surface(state.compositor);
```

Before we can present it, we must first attach a source of pixels to it: a
`wl_buffer`.

在真正显示它之前，需要首先为它添加一个像素源：一个 `wl_buffer`。

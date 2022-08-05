---
layout: post
title:  "8.2 Wayland book Frame callbacks"
date:   2022-08-05 13:55:29 +0800
categories: jekyll update
---
# Frame callbacks

The simplest way to update your surface is to simply render and attach new
frames when it needs to change. This approach works well, for example, with
event-driven applications. The user presses a key and the textbox needs to be
re-rendered, so you can just re-render it immediately, damage the appropriate
area, and attach a new buffer to be presented on the next frame.

更新表面的最简单方法是在需要更改时渲染新的一帧并附加到 surface。 例如，这种方法
适用于事件驱动的应用程序。 用户按下一个键，文本框需要重新渲染，所以你可以立即重新
渲染它，标记损坏的区域，并附加一个新的缓冲区以在下一帧呈现。

However, some applications may want to render frames continuously. You might be
rendering frames of a video game, playing back a video, or rendering an
animation. Your display has an inherent *refresh rate*, or the fastest rate at
which it's able to display updates (generally this is a number like 60 Hz, 144
Hz, etc). It doesn't make sense to render frames any faster than this, and doing
so would be a waste of resources &mdash; CPU, GPU, even the user's battery. If 
you send several frames between each display refresh, all but the last will be
discarded and have been rendered for naught.

但是，某些应用程序可能希望连续渲染。 比方说正在渲染视频游戏、播放视频或动画。显示器
具有固有的 *刷新率*，或者以最高速率刷新显示器（通常是 60 Hz、144 Hz 等数字）。 以比这更快
的速度渲染帧是没有意义的，这样做会浪费资源——CPU、GPU，甚至用户的电池。 如果你在每次显示器
刷新之间发送多个帧，那么除了最后一个帧之外的所有帧都将（白白的浪费了 gpu 做渲染并）被丢弃。
 
Additionally, the compositor might not even want to show new frames for you.
Your application might be off-screen, minimized, or hidden behind other windows;
or only a small thumbnail of your application is being shown, so they might want
to render you at a slower framerate to conserve resources. For this reason, the
best way to continuously render frames in a Wayland client is to let the
compositor tell you when it's ready for a new frame: using *frame callbacks*.

此外，合成器甚至可能不想为你（的应用程序）显示新的帧。（比方说）应用程序可能在屏幕外、
最小化或隐藏在其他窗口后面；或者只显示应用程序的一个小缩略图，（基于种种情况）他们（compositor）
可能希望以较慢的帧速率呈现以节省资源。出于这个原因，在 Wayland 客户端中连续渲染帧的最佳
方法是让合成器告诉你何时准备好接收新帧：使用 *frame callback*。

```xml
<interface name="wl_surface" version="4">
  <!-- ... -->

  <request name="frame">
    <arg name="callback" type="new_id" interface="wl_callback" />
  </request>

  <!-- ... -->
</interface>
```

This request will allocate a `wl_callback` object, which has a pretty simple
interface:

这个请求将创建一个 `wl_callback` 对象，它有一个非常简单的接口：

```xml
<interface name="wl_callback" version="1">
  <event name="done">
    <arg name="callback_data" type="uint" />
  </event>
</interface>
```

When you request a frame callback on a surface, the compositor will send a
`done` event to the callback object once it's ready for a new frame for this
surface. In the case of `frame` events, the `callback_data` is set to the
current time in millisecond, from an unspecified epoch. You can compare this
with your last frame to calculate the progress of an animation or to scale input
events.[^1]

当你为你的 surface 请求了一个帧回调，合成器会在它准备好接收对应 surface 的一个新帧
的时候，对这个 callback 对象发送 `done` 事件。关于这个 `frame` 事件，参数 `callback_data`
的内容是单位为毫秒的从Server创建开始计时的当前时间。你可以和之前一帧对比，计算出准确的
动画进度或输入的缩放事件。[^1]

With frame callbacks in our toolbelt, why don't we update our application from
chapter 7.3 so it scrolls a bit each frame? Let's start by adding a little bit
of state to our `client_state` struct:

有了帧回调这个工具，让我们更新第 7.3 章的应用程序，让他能够每帧滑动一点。让我们从往
`client_state` 结构体里增加state开始：

```diff
--- a/client.c
+++ b/client.c
@@ -71,6 +71,8 @@ struct client_state {
 	struct xdg_surface *xdg_surface;
 	struct xdg_toplevel *xdg_toplevel;
+	/* State */
+	float offset;
+	uint32_t last_frame;
 };
 
 static void wl_buffer_release(void *data, struct wl_buffer *wl_buffer) {
```

Then we'll update our `draw_frame` function to take the offset into account:

接下来我们更新 `draw_frame` 函数，将偏移动作考虑在内：

```diff
@@ -107,9 +109,10 @@ draw_frame(struct client_state *state)
 	close(fd);
 
 	/* Draw checkerboxed background */
+	int offset = (int)state->offset % 8;
 	for (int y = 0; y < height; ++y) {
 		for (int x = 0; x < width; ++x) {
-			if ((x + y / 8 * 8) % 16 < 8)
+			if (((x + offset) + (y + offset) / 8 * 8) % 16 < 8)
 				data[y * width + x] = 0xFF666666;
 			else
 				data[y * width + x] = 0xFFEEEEEE;
```

In the `main` function, let's register a callback for our first new frame:

在 `main` 函数中，我们为第一帧注册了帧回调：

```diff
@@ -195,6 +230,9 @@ main(int argc, char *argv[])
 	xdg_toplevel_set_title(state.xdg_toplevel, "Example client");
 	wl_surface_commit(state.wl_surface);
 
+	struct wl_callback *cb = wl_surface_frame(state.wl_surface);
+	wl_callback_add_listener(cb, &wl_surface_frame_listener, &state);
+
 	while (wl_display_dispatch(state.wl_display)) {
 		/* This space deliberately left blank */
 	}
```

Then implement it like so:

关于 callback 的实现如下：

```diff
@@ -147,6 +150,38 @@ static const struct xdg_wm_base_listener xdg_wm_base_listener = {
 	.ping = xdg_wm_base_ping,
 };
 
+static const struct wl_callback_listener wl_surface_frame_listener;
+
+static void
+wl_surface_frame_done(void *data, struct wl_callback *cb, uint32_t time)
+{
+	/* Destroy this callback */
+	wl_callback_destroy(cb);
+
+	/* Request another frame */
+	struct client_state *state = data;
+	cb = wl_surface_frame(state->wl_surface);
+	wl_callback_add_listener(cb, &wl_surface_frame_listener, state);
+
+	/* Update scroll amount at 24 pixels per second */
+	if (state->last_frame != 0) {
+		int elapsed = time - state->last_frame;
+		state->offset += elapsed / 1000.0 * 24;
+	}
+
+	/* Submit a frame for this event */
+	struct wl_buffer *buffer = draw_frame(state);
+	wl_surface_attach(state->wl_surface, buffer, 0, 0);
+	wl_surface_damage_buffer(state->wl_surface, 0, 0, INT32_MAX, INT32_MAX);
+	wl_surface_commit(state->wl_surface);
+
+	state->last_frame = time;
+}
+
+static const struct wl_callback_listener wl_surface_frame_listener = {
+	.done = wl_surface_frame_done,
+};
+
 static void
 registry_global(void *data, struct wl_registry *wl_registry,
 		uint32_t name, const char *interface, uint32_t version)
```

Now, with each frame, we'll

现在，对于每一帧，我们会

1. Destroy the now-used frame callback.
2. Request a new callback for the next frame.
3. Render and submit the new frame.

1. 销毁现在使用的帧回调。
2. 请求下一帧的新回调。
3. 渲染并提交新帧。

The third step, broken down, is:

第三步分解为：

1. Update our state with a new offset, using the time since the last frame to
   scroll at a consistent rate.
2. Prepare a new `wl_buffer` and render a frame for it.
3. Attach the new `wl_buffer` to our surface.
4. Damage the entire surface.
5. Commit the surface.

1. 使用新的偏移量更新我们的状态，根据上一帧传来的时间戳以平滑一致的速度滚动 surface。
2. 准备一个新的，已经完成渲染帧的 `wl_buffer`。
3. 将新的 `wl_buffer` 附加到我们的 surface。
4. Damage 整个表面。
5. 提交表面。

Steps 3 and 4 update the *pending* state for the surface, giving it a new buffer
and indicating the entire surface has changed. Step 5 commits this pending
state, applying it to the surface's current state, and using it on the following
frame. Applying this new buffer atomically means that we never show half of the
last frame, resulting in a nice tear-free experience. Compile and run the
updated client to try it out for yourself!

第 3 步和第 4 步更新表面的 *pending* 状态，为其提供一个新缓冲区并指示整个表面已更改。 
步骤 5 提交此挂起状态，将其应用于表面的当前状态，并在下一帧中使用它。 以原子方式应用
这个新缓冲区意味着我们永远不会显示最后一帧的一半，从而能够获得不错的无撕裂体验。 编译
并运行更新后的客户端，亲自尝试一下！

[^1]: Want something more accurate? In chapter 12.1 we talk about a protocol extension which tells you with nanosecond resolution exactly when each frame was presented to the user.
[^1]：想要更准确的东西？ 在第 12.1 章中，我们讨论了一个协议扩展，它以纳秒级的时间戳准确地告诉你每一帧何时呈现给用户。

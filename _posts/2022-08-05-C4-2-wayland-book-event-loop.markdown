---
layout: post
title:  "4.2 Wayland book Incorporating an event loop"
date:   2022-08-05 11:08:29 +0800
categories: jekyll update
---
# Incorporating an event loop

libwayland provides its own event loop implementation for Wayland servers to
take advantage of, but the maintainers have acknowledged this as a design
overstep. For clients, there is no such equivalent. However, the Wayland server
event loop is useful enough, even if it's out-of-scope.

libwayland 实现了自己的 event loop， 并将它提供给 Wayland server 使用。而对于 client，
则没有提供这样的机制。虽然开发维护者也认为这在架构的整体设计上是越界的。但是，
Wayland server 的 event loop 真的很有用~

## Wayland server event loop

Each `wl_display` created by libwayland-server has a corresponding
`wl_event_loop`, which you may obtain a reference to with
`wl_display_get_event_loop`. If you're writing a new Wayland compositor, you
will likely want to use this as your only event loop. You can add file
descriptors to it with `wl_event_loop_add_fd`, and timers with
`wl_event_loop_add_timer`. It also handles signals via
`wl_event_loop_add_signal`, which can be pretty convenient.

libwayland-server 创建的每个 `wl_display` 都有一个对应的 `wl_event_loop`，你可以使用
`wl_display_get_event_loop` 获得对 loop 的引用。 如果你正在编写一个新的 Wayland compositor，
可以通过这个接口获得唯一的 event loop。 （有了 loop 以后）可以使用 `wl_event_loop_add_fd`
为 loop 增加对（对应的）fd 的监听，使用 `wl_event_loop_add_timer` 添加定时器。甚至可以通过
`wl_event_loop_add_signal` 添加需要处理的信号，总之，非常方便。

With the event loop configured to your liking to monitor all of the events your
compositor has to respond to, you can process events and dispatch Wayland
clients all at once by calling `wl_display_run`, which will process the event
loop and block until the display terminates (via `wl_display_terminate`). Most
Wayland compositors which were built from the ground-up with Wayland in mind (as
opposed to being ported from X11) use this approach.

你可以随心所欲的配置自己的 event-loop，监听自己的 compositor 必须响应的所有 event。
可以通过调用 `wl_display_run` 一次性完成处理 event，派发消息至 Wayland client。
run 函数会一直不知疲倦的循环工作，直至死亡（`wl_display_terminate`）。大多数从头开始构建的
Wayland compositor（而不是从 X11 移植）都是使用这种方法实现的（消息处理循环机制）。

However, it's also possible to take the wheel and incorporate the Wayland
display into your own event loop. `wl_display` uses the event loop internally
for processing clients, and you can choose to either monitor the Wayland event
loop on your own, dispatching it as necessary, or you can disregard it
entirely and manually process client updates. If you wish to allow the Wayland
event loop to look after itself and treat it as subservient to your own event
loop, you can use `wl_event_loop_get_fd` to obtain a [poll][poll]-able file
descriptor, then call `wl_event_loop_dispatch` to process events when activity
occurs on that file descriptor. You will also need to call
`wl_display_flush_clients` when you have data which needs writing to clients.

当然，也是可以将 Wayland display 和循环放到你自己的 event loop里面。`wl_display` 在内部使用
event loop 来处理 client 的 request，你可以选择自己监控 Wayland event loop，必要时调度它；或者完全忽视它，
自己手动处理 client 的 request 完成更新。如果你希望 Wayland event loop 自行处理相关任务，并将他变为你自己
event loop 中的一部分，可以使用 `wl_event_loop_get_fd` 来获取一个（基于 Wayland event loop） [poll][poll]-able 的 fd，
之后此 fd 发生活动时，调用 `wl_event_loop_dispatch` 处理事件。当需要将数据写入 client 时，还需要调用 `wl_display_flush_clients`。

[poll]: https://pubs.opengroup.org/onlinepubs/009695399/functions/poll.html

## Wayland client event loop

libwayland-client, on the other hand, does not have its own event loop. However,
since there is only generally one file descriptor, it's easier to manage
without. If Wayland events are the only sort which your program expects, then
this simple loop will suffice:

另一边，libwayland-client 没有自己的 event loop。然而，由于通常只有一个 fd（也就是和 sever 交互的 socket），
不使用 event loop 明显更容易管理。如果Wayland 的 event 事件是 client 所期望的唯一消息来源，那么调用如下的简单的循环就足够了:

```c
while (wl_display_dispatch(display) != -1) {
    /* This space deliberately left blank */
}
```

However, if you have a more sophisticated application, you can build your own
event loop in any manner you please, and obtain the Wayland display's file
descriptor with `wl_display_get_fd`. Upon `POLLIN` events, call
`wl_display_dispatch` to process incoming events. To flush outgoing requests,
call `wl_display_flush`.

但是，如果你有更复杂的应用程序，你完全可以按照自己喜欢的任何方式构建自己的 event loop，
并使用 `wl_display_get_fd` 获取 Wayland display 的 fd。在 POLLIN 事件上，调用 `wl_display_dispatch`
来处理传入的 event。要刷新传出去的 request，请调用 `wl_display_flush`。

## Almost there!

At this point you have all of the context you need to set up a Wayland
display and process events and requests. The only remaining step is to allocate
objects to chat about with the other side of your connection. For this, we use
the registry. At the end of the next chapter, we will have our first useful
Wayland client!

就目前为止，你已拥有设置 Wayland display 和处理 event 和 request 所需的所有环境相关的方法函数。
剩下的唯一步骤是分配 object 以与另一边进行连接，交互。我们使用 registry 实现上述需求。
在下一章的结尾，我们将拥有第一个可用的 Wayland 客户端！

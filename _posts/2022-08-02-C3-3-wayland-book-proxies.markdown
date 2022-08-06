---
layout: post
title:  "3.3 Wayland book Libwayland in depth:Proxies & resources"
date:   2022-08-02 21:21:29 +0800
categories: jekyll update
---
# Proxies & resources

An *object* is an entity which known to both the client and server that has some
state, changes to which are negotiated over the wire. On the client side,
libwayland refers to these objects through the `wl_proxy` interface. These are a
concrete C-friendly "proxy" for the abstract object, and provides functions
which are indirectly used by the client to marshall requests into the wire
format. If you review the `wayland-client-core.h` file, you'll find a few
low-level functions for this purpose. Generally, you don't use these directly.

对象 *object* 是 client 端和 server 端相互关联的一个业务实体（entity），对象上面存储了
一些状态（state，也可以理解为参数），client 端和 server 端通过网络协商来改变两边的对象
状态值。在 client 端，libwayland 通过 `wl_proxy` 接口来引用这些对象。这些接口以具体的 C
语言形式实现，专门用来 "代理" 各种抽象对象的（比如前文所说的 primitive 中的 `display/surface/buffer` 等对象），
并为 client 提供了一些底层函数，专门用于将请求（request）打包成线上传输格式（wire format）
的数据。如果你查看 `wayland-client-core.h` 文件，你就会看到这些底层函数。一般来说，不会
直接使用这些函数。

On the server, objects are abstracted through `wl_resource`, which is fairly
similar, but have an extra degree of complexity &mdash; the server has to track
which object belongs to which client. Each `wl_resource` is owned by a single 
client. Aside from this, the interface is much the same, and provides low-level
abstraction for marshalling events to send to the associated client. You will
use `wl_resource` directly on a server more often than you'll use directly
interface with `wl_proxy` on a client. One example of such a use is to obtain a
reference to the `wl_client` which owns a resource that you're manipulating
out-of-context, or send a protocol error when the client attempts an invalid
operation.

在 server 端，object 由 `wl_resource` 结构体表示，这和前面 client 端采用 `wl_proxy` 结构体
来表示类似，但有一点不同 &mdash; server 端必须记录某个对象具体属于哪个 client 进程。每个 `wl_resource`
都隶属于一个独立的 client 进程。除了刚才说的那一点不同之外，server 端的接口跟 client 端基本
相同，同时还提供了底层抽象层，用于将事件（event）打包并发送给对应的 client 端。你在 server 端
直接使用 `wl_resource` 会比在 client 端直接使用`wl_proxy` 更频繁。举一个直接使用 `wl_resource` 的例子：
当你想要在 server 端操作某些脱离了当前上下文的资源时，你可以通过 `wl_resource` 来获得这些资源
所对应的 `wl_client` 的引用（通过 `wl_client` 你就可以获得这些资源了），或者在 client 端试图
进行无效操作时发送一个协议错误的消息事件给 client 端。

Another level up is another set of higher-level interfaces, which most Wayland
client & server code interacts with to accomplish a majority of their tasks.

另外还有一个更上层的接口集合（封装了 client 端和 server 端各种底层函数），用来与 wayland client
和 server 端代码进行交互，从而完成大部分的任务。

原文链接:https://wayland-book.com/

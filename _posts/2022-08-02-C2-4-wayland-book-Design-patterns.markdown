---
layout: post
title:  "2.4 Wayland book Protocol design:Protocol design patterns"
date:   2022-08-02 21:11:29 +0800
categories: jekyll update
---
# Protocol design patterns

There are some key concepts which have been applied in the design of most
Wayland protocols that we should briefly cover here. These patterns are found
throughout the high-level Wayland protocol and protocol extensions (well, in the
Wayland protocol, at least[^1]). If you find yourself writing your own
protocol extensions, you'd be wise to apply these patterns yourself.

在大多数 Wayland 协议的设计中应用了一些关键概念，我们在这里简要介绍一下。
这些模式准则可以在高级 Wayland 协议和协议扩展中找到（至少在 Wayland 协议中可以找到[^1]）。
如果你发现自己正在编写自己的协议扩展，那么你最好借鉴这些模式准则。

## Atomicity

The most important of the Wayland protocol design patterns is *atomicity*. A
stated goal of Wayland is "every frame is perfect". To this end, most interfaces
allow you to update them transactionally, using several requests to build up a
new representation of its state, then committing them all at once. For example,
there are several properties that may be configured on a `wl_surface`:

Wayland 协议设计模式中最重要的是原子性。Wayland 的一个既定目标是 "每一帧都是完美的"。
为此，大多数接口允许（同一帧）多次调用请求，来不断构建新的展现方式，然后一次性提交。例如，
可以在 `wl_surface` 上的配置如下属性：

- An attached pixel buffer
- A damaged area that needs to be redrawn
- A region defined as opaque, for optimization purposes
- A region where input events are acceptable
- A transformation, like rotating 90 degrees
- A buffer scale, used for HiDPI

-	附加的像素缓冲区
-	需要重新绘制的损坏区域
-	定义为不透明的区域，用于优化目的
-	可接受输入事件的区域
-	一种变换，比如旋转 90 度
-	缓冲比例，用于 HiDPI


The interface includes separate requests for configuring each of these, but
these are applied to a *pending* state. Only when the **commit** request is sent
does the pending state get merged into the *current* state, allowing you to
atomically update all of these properties within a single frame. Combined with a
few other key design decisions, this allows Wayland compositors to render
everything perfectly in every frame &mdash; no tearing or partially updated 
windows, just every pixel in its place and every place in its pixel.

接口为这些请求提供了单独的配置，但是这些请求仅被应用于 *挂起* 状态。只有当 **提交** 请求
被发送时，挂起状态才会合并到 *当前* 状态，从而使你可以在单帧内原子地更新所有这些属性。结合
其他一些关键的设计决策，这使得 Wayland compositor 能够在每一帧中完美地渲染所有内容 &mdash; 
没有撕裂或仅仅部分更新的窗口，每个像素都在它应该在的位置上。

## Resource lifetimes

Another important design pattern is avoiding a situation where the server or
client is sending events or requests that pertain to an invalid object. For this
reason, interfaces which define resources that have finite lifetimes will often
include requests and events through which the client or server can state their
intention to no longer send requests or events for that object. Only once both
sides have agreed to this &mdash; asynchronously &mdash; do they destroy the 
resources they allocated for that object.

另一个重要的设计准则是避免服务端或客户端发送基于无效对象的事件或请求。出于这个原因，
接口定义的请求和事件具有有限生命周期，客户端或服务端可以对相应接口，显示声明他们不再
为该对象发送请求或反馈事件。只有在双方同意这一点后 &mdash; 异步地 &mdash; 他们才会
销毁他们为该对象分配的资源。

Wayland is a fully asynchronous protocol. Messages are guaranteed to arrive in
the order they were sent, but only with respect to one sender. For example, the
server may have several input events queued up when the client decides to
destroy its keyboard device. The client must correctly deal with events for an
object it no longer needs until the server catches up. Likewise, had the client
queued up some requests for an object before destroying it, it would have had to
send these requests in the correct order so that the object is no longer used
after the client agreed it had been destroyed.

Wayland 是一个完全异步的协议。消息能够保证按发送顺序到达，但仅限于同一个发送者。例如，
当客户端决定销毁其键盘设备时，服务端可能有多个输入事件排队。客户端必须正确处理它不再
需要的对象的事件，直到服务端赶上（直到服务端和客户端接近同步）。同样，如果客户端在销
毁对象之前将一些请求排入队列，则必须以正确的顺序发送这些请求，以便客户端在服务端同意
对象已被销毁后不再使用该对象。（注意这一点，这是我们 client 经常 coredump 的原因）

## Versioning

There are two versioning models in use in Wayland protocols: unstable and
stable. Both models only allow for backwards-compatible changes, but when a
protocol transitions from unstable to stable, one last breaking change is
permitted. This gives protocols an incubation period during which we can test
them in practice, then apply our insights in one last big set of breaking
changes to make a protocol that should stand the test of time[^2].

Wayland 协议中使用了两种版本控制模型：不稳定和稳定。两种版本模型都只允许向后兼容的更
改，但是当协议从不稳定过渡到稳定时，允许进行最后一次重大更改。这为协议提供了一个潜伏
期，在此期间我们可以在实践中对其进行测试，然后将我们的见解应用于最后一组重大更改，以
制定经得起时间考验的协议[^2]。

To make a backwards-compatible change, you may only add new events or requests
to the end of an interface, or new members to the end of an enum. Additionally,
each implementation must limit itself to using only the messages supported by
the other end of the connection. We'll discuss in chapter 5 how we establish
which versions of each interface are in use by each party.

要进行向后兼容的更改，你只能在接口的末尾添加新事件或请求，或者在枚举的末尾添加新成员。
此外，每个实现必须限制为仅使用连接另一端支持的消息。我们将在第 5 章讨论，如何确定各方
正在使用的每个接口的版本。

[^1]: Except for that one interface. Look, at least we tried, right?

[^1]: 除了那个接口。看，至少我们试过了，对吧？

[^2]: Note that many useful protocols are still unstable at the time of writing. They may be a little kludgy, but they still see widespread use, which is why backwards compatibility is important. When promoting a protocol from unstable to stable, it's done in a way that allows software to support both the unstable and stable protocols simultaneously, allowing for a smoother transition.

[^2]: 请注意，在撰写本文时，许多有用的协议仍然不稳定。它们可能有点笨拙，但它们仍然被广泛使用，这就是向后兼容性很重要的原因。将协议从不稳定升级到稳定时，其方式是允许软件同时支持不稳定和稳定协议，从而实现更平滑的过渡。

原文链接:https://wayland-book.com/

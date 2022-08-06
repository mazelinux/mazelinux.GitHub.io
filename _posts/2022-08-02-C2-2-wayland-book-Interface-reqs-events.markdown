---
layout: post
title:  "2.2 Wayland book Protocol design:Interface, requests, events"
date:   2022-08-02 21:06:29 +0800
categories: waylandbook
---
# Interfaces, requests, and events

The Wayland protocol works by issuing *requests* and *events* that act on
*objects*. Each object has an *interface* which defines what requests and events
are possible, and the *signature* of each. Let's consider an example interface:
`wl_surface`.

Wayland 协议通过发出作用于 *对象* 的 *请求* 和 *事件* 来工作。每个对象都有一个 *接口*，
定义了可用的请求和事件，以及对应的签名。让我们参照一个示例接口：`wl_surface`。

## Requests

A surface is a box of pixels that can be displayed on-screen. It's one of the
primitives we build things like application windows out of. One of its
*requests* is "damage", which the client uses to indicate that some part of the
surface has changed and needs to be redrawn. Here's an annotated example of a
"damage" message on the wire (in hexadecimal):

Surface 是满满一盒可以显示在屏幕上的像素。它是我们构建诸如应用程序窗口的原语之一。
它的请求之一是 "damage"，客户端使用它来表明 surface 的某些部分已更改并需要重新绘制。
这是 wire 上带有 "damage" 消息的注释示例（十六进制）：

    0000000A    Object ID (10)
    00180002    Message length (24) and request opcode (2)
    00000000    X coordinate (int): 0
    00000000    Y coordinate (int): 0
    00000100    Width        (int): 256
    00000100    Height       (int): 256

This is a snippet of a session &mdash; the surface was allocated earlier and 
assigned an ID of 10. When the server receives this message, it looks up the 
object with ID 10 and finds that it's a `wl_surface` instance. Knowing this, 
it looks up the signature for the request with opcode 2. It then knows to expect 
four integers as the arguments, and it can decode the message and dispatch it 
for processing internally.

这是一个会话的片段 &mdash; surface 早先被分配了 ID 为 10。当服务器收到这个消息时，
它查找 ID 为 10 的对象并发现它是一个 `wl_surface` 实例。知道了这一点，它使用操作码
2 查找请求的签名。然后它知道需要四个整数作为参数，随后它可以在内部处理消息解码并
将其（消息）派发出去。

## Events

Requests are sent from the client to the server. The server can also send
messages back &mdash; events. One event that the server can send regarding a
`wl_surface` is "enter", which it sends when that surface is being displayed on
a specific output (the client might respond to this, for example, by adjusting
its scale factor for a HiDPI display). Here's an example of such a message:

请求从客户端发送到服务端。服务端还可以回发消息 &mdash; 事件。服务端可以发送关于 wl_surface
的 "输入" 事件，该事件会在该 surface 显示在特定 Output（因为这个output未必是你的显示器，可也能是离屏渲染） 
上以后，发送（客户端可能对此做出响应，例如，通过调整其 HiDPI 显示的比例因子）。以下
是此类消息的示例：

    0000000A    Object ID (10)
    000C0000    Message length (12) and event opcode (0)
    00000005    Output (object ID): 5

This message references another object, by its ID: the `wl_output` object which
the surface is being shown on. The client receives this and dances to a similar
tune as the server did. It looks up object 10, associates it with the
`wl_surface` interface, and looks up the signature of the event corresponding to
opcode 0. It decodes the rest of the message accordingly (looking up the
`wl_output` with ID 5 as well), then dispatches it for processing internally.

此消息通过 ID 引用了另一个对象：`wl_output`，它用于 surface 对象的显示。客户端收到此信息后，
与服务端处理基本一致。它查找ID 为 10 的对象，将其与 `wl_surface` 接口相关联 ，并查找签名，
找对应于操作码 0 的事件。相应地解码消息的其余部分（也查找ID 为 5的 `wl_output`），
然后将其内部派发以进行处理。

## Interfaces

The interfaces which define the list of requests and events, the opcodes
associated with each, and the signatures with which you can decode the messages 
&mdash; are agreed upon in advance. I'm sure you're dying to know how &mdash; 
simply turn the page to end the suspense.

接口定义请求和事件，相关的操作码以及可以用来解码消息的签名 &mdash; 这些都是事前协定的。
我相信你肯定很想知道原理 &mdash; 只需翻页即可解答你的疑问。

原文链接:https://wayland-book.com/

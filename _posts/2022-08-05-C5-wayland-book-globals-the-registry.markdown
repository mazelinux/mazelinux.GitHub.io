---
layout: post
title:  "5 Wayland book Globals & the registry"
date:   2022-08-05 11:17:29 +0800
categories: waylandbook
---
# Globals & the registry

If you'll recall from chapter 2.1, each request and event is associated with an
object ID, but thus far we haven't discussed how objects are created. When we
receive a Wayland message, we must know what interface the object ID represents
to decode it. We must also somehow negotiate available objects, the creation of
new ones, and the assigning of IDs to them, in some manner. On Wayland we solve
both of these problems at once &mdash; when we *bind* an object ID, we agree on 
the interface used for it in all future messages, and stash a mapping of object 
IDs to interfaces in our local state.

回想一下 2.1 章节的内容：每个 "请求" 和 "事件" 都会与一个 object ID 相关联，不过到目前为止，
我们都还没有讨论过这些 object 是如何创建出来的。当我们收到一个 Wayland 消息时，我们必须清楚该消息中的 object ID
具体对应的是哪个接口，这样才能对该消息进行正确地解码。我们还必须以某种方式协商哪些对象是已经存在且可用的，哪些是需要新创建的，
以及为这些对象分配新的 ID。在 Wayland 中我们同时解决了这两个问题 &mdash; 当我们 *bind*（绑定） 到某个 object ID 时，
我们也就确定了后续所有消息的操作接口，并且会在本地状态（local state）中保存一份 object ID 到 interface 的映射表。

In order to bootstrap these, the server offers a list of *global* objects. These
globals often provide information and functionality on their own merits, but
most often they're used to broker additional objects to fulfill various
purposes &mdash; such as the creation of application windows. These globals 
themselves also have their own object IDs and interfaces, which we have to 
assign and agree upon somehow.

为了实现这些操作，server 端为我们提供了一个 *全局对象* 列表（global object）。这些全局对象通常会根据它们自己的特点来提供
相应的信息和功能，但大多数情况下，它们用于代理其他对象，以实现更多的功能 &mdash; 例如创建应用程序窗口。同样的，这些全局对象本身
也有它们自己的 object ID 和 interface，因此我们仍然需要以某种方式来分配这些 ID，并通过这些 interface 来和 server 端进行交互。


With questions of hens and eggs no doubt coming to mind by now, I'll reveal the
secret trick: object ID 1 is already implicitly assigned to the `wl_display`
interface when you make the connection. As you'll recall the interface, take
note of the `wl_display::get_registry` request:

此时此刻，你一定想到了鸡和蛋的问题，别担心，我这就为你揭开谜底：当建立连接时，object ID 1 就已经被悄悄地
分配给了 *wl_display* 接口了。现在让我们一起来回顾一下 *wl_display* 这个接口吧，请注意留心 `wl_display::get_registry` 这个请求:

```xml
<interface name="wl_display" version="1">
  <request name="sync">
    <arg name="callback" type="new_id" interface="wl_callback" />
  </request>

  <request name="get_registry">
    <arg name="registry" type="new_id" interface="wl_registry" />
  </request>

  <!-- cotd -->
</interface>
```

The `wl_display::get_registry` request can be used to bind an object ID to the
`wl_registry` interface, which is the next one found in `wayland.xml`. Given
that the `wl_display` always has object ID 1, the following wire message ought
to make sense (in big-endian):

`wl_display::get_registry` 请求用于绑定一个 object ID 到 `wl_registry` 接口（该接口在 `wayland.xml` 
中的位置仅次于 `wl_display` 接口）。由于 `wl_display` 的 object ID 总是 1 ，因此下面的消息协议在实际案例中
是真实存在的（请注意，这里使用大端模式来表示）：

```
C->S    00000001 000C0001 00000002            .... .... ....
```

When we break this down, the first number is the object ID. The most significant
16 bits of the second number are the total length of the message in bytes, and
the least significant bits are the request opcode. The remaining words (just
one) are the arguments. In short, this calls request 1 (0-indexed) on object ID
1 (the `wl_display`), which accepts one argument: a generated ID for a new
object. Note in the XML documentation that this new ID is defined ahead of time
to be governed by the `wl_registry` interface:

让我们逐步分解这段字符，第一个数字是对象 ID。第二个数字的最高 16 位是消息的总长度（以字节为单位），
最低有效位是请求（request）的操作码。剩下的最后一个是参数。简而言之，这在对象 ID 1（wl_display）上调用请求 1（索引从 0 开始），
它接收一个参数：为新对象生成的 ID。请注意，在 XML 文档中，这个新 ID 是提前定义的，由 wl_registry 接口管理：

```xml
<interface name="wl_registry" version="1">
  <request name="bind">
    <arg name="name" type="uint" />
    <arg name="id" type="new_id" />
  </request>

  <event name="global">
    <arg name="name" type="uint" />
    <arg name="interface" type="string" />
    <arg name="version" type="uint" />
  </event>

  <event name="global_remove">
    <arg name="name" type="uint" />
  </event>
</interface>
```

It is this interface which we'll discuss in the following chapters.

我们将在接下来的章节中讨论这个接口。

原文链接:https://wayland-book.com/

---
layout: post
title:  "3.4 Wayland book Libwayland in depth:Interfaces & listeners"
date:   2022-08-02 21:26:29 +0800
categories: jekyll update
---
# Interfaces & listeners

Finally, we reach the summit of libwayland's abstractions: interfaces and
listeners. The ideas discussed in previous chapters &mdash; `wl_proxy` and
`wl_resource`, and the primitives &mdash; are singular implementations which 
live in libwayland, and they exist to provide support to this layer. When you
run an XML file through wayland-scanner, it generates *interfaces* and
*listeners*, as well as glue code between them and the low-level wire protocol
interfaces, all specific to each interface in the high-level protocols.

最后，我们来到了 libwayland 抽象类的集大成者：interface（接口） 和 listener（监听器）。
在前几章我们讨论过的 &mdash; `wl_proxy`、`wl_resource` 以及primitive mdash; 这些都是
libwayland 中的单个实现，它们的存在是为了给这一层的接口和监听器提供支持的。当你通过
`wayland-scanner` 解析一个 XML 文件时，它会生成各种各样的 *interface* 和 *listener*，
以及它们与底层协议接口之间的粘合代码，所有这些粘合代码都对应到上层协议的每一个接口。

Recall that each actor on a Wayland connection can both receive and send
messages. A client is listening for events and sending requests, and a server
listens for requests and sends events. Each side listens for the messages of the
other using an aptly-named `wl_listener`. Here's an example of this interface:

回顾一下，Wayland 连接上的每个角色都可以接收和发送消息。一个 client 在监听事件（event）并
发送请求（request），而一个 server 则在监听请求并发送事件。每一方都使用一个类似命名为
`wl_listener` 的监听器来监听对方的消息。下面是该接口的示例代码：

```c
struct wl_surface_listener {
	/** surface enters an output */
	void (*enter)(void *data,
		      struct wl_surface *wl_surface,
		      struct wl_output *output);

	/** surface leaves an output */
	void (*leave)(void *data,
		      struct wl_surface *wl_surface,
		      struct wl_output *output);
};
```

This is a client-side listener for a `wl_surface`. The XML that wayland-scanner
uses to generate this is:

这是一个 `wl_surface` 对象上的 client 端消息监听器。`wayland-scanner` 生成上述监听器
代码的 XML 如下：

```xml
<interface name="wl_surface" version="4">
  <event name="enter">
    <arg name="output"
      type="object"
      interface="wl_output"/>
  </event>

  <event name="leave">
    <arg name="output"
      type="object"
      interface="wl_output"/>
  </event>
  <!-- additional details omitted for brevity -->
</interface>
```

It should be fairly clear how these events become a listener interface. Each
function pointer takes some arbitrary user data, a reference to the resource
which the event pertains to, and the arguments to that event. We can bind a
listener to a `wl_surface` like so:

从上面的 XML 文件中我们可以清晰的看出，这些 event 是如何转换成单个 listener 中的函数
接口的。每个函数指针接收一个用户自定义的 data 参数、一个指向该 event 所属资源的引用
（即 `wl_surface` 指针），以及该 event 在 XML 中所定义的 arg 参数。我们可以像下面这样
将一个 listener 绑定到 `wl_surface` 上：

```c
static void wl_surface_enter(void *data,
        struct wl_surface *wl_surface, struct wl_output *output) {
    // ...
}

static void wl_surface_leave(void *data,
        struct wl_surface *wl_surface, struct wl_output *output) {
    // ...
}

static const struct wl_surface_listener surface_listener = {
    .enter = wl_surface_enter,
    .leave = wl_surface_leave,
};

// ...cotd...

struct wl_surface *surf;
wl_surface_add_listener(surf, &surface_listener, NULL);
```

The `wl_surface` interface also defines some requests that the client can make
for that surface:

`wl_surface` 接口还为 client 定义了许多针对该 surface 的请求操作：

```xml
<interface name="wl_surface" version="4">
  <request name="attach">
    <arg name="buffer"
      type="object"
      interface="wl_buffer"
      allow-null="true"/>
    <arg name="x" type="int"/>
    <arg name="y" type="int"/>
  </request>
  <!-- additional details omitted for brevity -->
</interface>
```

wayland-scanner generates the following prototype, as well as glue code which
marshalls this message.

`wayland-scanner` 将为我们生成如下函数原型，以及将该消息打包的底层粘合代码。

```c
void wl_surface_attach(struct wl_surface *wl_surface,
    struct wl_buffer *buffer, int32_t x, int32_t y);
```

The server-side code for interfaces and listeners is identical, but reversed 
&mdash; it generates listeners for requests and glue code for events. When 
libwayland receives a message, it looks up the object ID, and its interface, 
then uses that to decode the rest of the message. Then it looks for listeners on
this object and invokes your functions with the arguments to the message.

server 端的 interface 和 listener 代码跟 client 端是相同的，但是作用刚好相反 &mdash; 
它为 request 生成监听器，为 event 生成粘合代码。当 libwayland 收到一个消息时，它会查找
对象的 ID 和它属于 XML 协议中的哪个 interface（比如上面的 `wl_surface`），然后用找到的
interface 来解码消息的其余部分。接着，它会在这个对象上查找监听器，并使用消息参数来调用
你的回调函数。

That's all there is to it! It took us a couple of layers of abstraction to get
here, but you should now understand how an event starts in your server code,
becomes a message on the wire, is understood by the client, and dispatched to
your client code. There remains one unanswered question, however. All of this
presupposes that you already have references to Wayland objects. How do you get
those?

这就是 libwayland 的全部内容! 我们拨开了层层面纱终于来到了这里，而你现在也应该明白一个 event
是如何从你的 server 端代码开始，变成网络上的消息，接着被 client 解析，并被派发到你的 client
代码逻辑中。然而，还有一个问题没有得到解答：所有这些分析，都是建立在你已经有了 Wayland objects
的引用这一前提上。那么你是如何获得这些对象（object）的呢？

原文链接:https://wayland-book.com/

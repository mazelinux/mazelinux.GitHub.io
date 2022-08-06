---
layout: post
title:  "5.1 Wayland book Binding to globals"
date:   2022-08-05 11:20:29 +0800
categories: jekyll update
---
# Binding to globals

Upon creating a registry object, the server will emit the `global` event for
each global available on the server. You can then bind to the globals you
require.

在我们创建好注册表对象（registry object）后，server 端会即刻为每个可用的全局对象发送 `global` event（到 client）
，这样（client）就可以绑定到对应的全局对象上。

This process of taking a known object and assigning it an ID is called
*binding* the object. Once the client binds to the registry like this, the
server emits the `global` event several times to advertise which interfaces it
supports. Each of these globals is assigned a unique `name`, as an unsigned
integer. The `interface` string maps to the name of the interface found in the
protocol: `wl_display` from the XML above is an example of such a name. The
version number is also defined here &mdash; for more information about interface
versioning, see appendix C.

获取已知对象并为其分配 ID 的过程称为 *绑定* (binding)对象。一旦 client 像这样绑定到 register 对象，
server 就会多次发出 `global` 事件来表明它支持哪些接口。其中的每一个全局对象都被分配了一个唯一的无符号整数 `name` ，
`interface` 字符串对应了协议中接口名称：之前的 XML 文件中 `wl_display` 就是一个例子。还有版本号（version）也是在这里定义的 &mdash; 
有关接口版本控制的更多信息，请参见附录 C。

To bind to any of these interfaces, we use the bind request, which works
similarly to the magical process by which we bound to the `wl_registry`. For
example, consider this wire protocol exchange:

我们使用 'bind' request 来完成接口的绑定，它的工作原理和我们绑定 `wl_registry` 是一致的。举个例子，观察以下网络协议交互
内容：

```
C->S    00000001 000C0001 00000002            .... .... ....

S->C    00000002 001C0000 00000001 00000007   .... .... .... ....
        776C5f73 686d0000 00000001            wl_s hm.. ....
        [...]

C->S    00000002 00100000 00000001 00000003   .... .... .... ....
```

The first message is identical to the one we've already dissected. The second
one is an event from the server: object 2 (which the client assigned the
`wl_registry` to in the first message) opcode 0 ("global"), with arguments 1,
"wl_shm", and 1 &mdash; respectively the name, interface, and version of this 
global. The client responds by calling opcode 0 on object ID 2 
(`wl_registry::bind`) and assigns object ID `3` to global name `1` &mdash; 
*binding* to the `wl_shm` global. Future events and requests for this object
are defined by the `wl_shm` protocol, which you can find in `wayland.xml`.

第一条消息我们前面已经分析过了。第二条消息是来自 server 端返回的事件：对象 2（也就是在第一条消
息中由 client 分配的 `wl_registry` 接口所绑定的对象）、[001C是消息长度]操作码 0（即 global 事件），
接下来是对应的整形参数 1、"wl_shm" 字符串以及整数 1 &mdash; 它们分别对应全局对象的 name、interface 和版本号。
接着，client 端通过调用 object ID 为 2 的对象上的执行操作码 0（即 `wl_registry::bind` 请求），
并使用 global name `1` 来代指 object ID `3` 这个对象， 即 *绑定*（binding） 到 `wl_shm` 这个
全局对象上。此后该对象所有的事件和请求均由 `wl_shm` 协议定义，你可以在 `wayland.xml` 中找到相关内容。

Once you've created this object, you can utilize its interface to accomplish
various tasks &mdash; in the case of `wl_shm`, managing shared memory between 
the client and server. Most of the remainder of this book is devoted to 
explaining the usage of each of these globals.

Armed with this information, we can write our first useful Wayland client: one
which simply prints all of the globals available on the server.

一旦你成功创建了这个对象，你就可以利用它的接口来完成各种任务 &mdash; 例如在 `wl_shm` 这个案例中，你可以用它来管理
client 和 server 端之间的共享内存。其实本书绝大部分内容都是在讲解这些全局对象的用法。

Armed with this information, we can write our first useful Wayland client: one
which simply prints all of the globals available on the server.

有了这些信息，我们就可以写出第一个实用的 Wayland client 应用程序了：一个将 server 端所有可用的 global 都
打印出来的应用程序。

```c
#include <stdint.h>
#include <stdio.h>
#include <wayland-client.h>

static void
registry_handle_global(void *data, struct wl_registry *registry,
		uint32_t name, const char *interface, uint32_t version)
{
	printf("interface: '%s', version: %d, name: %d\n",
			interface, version, name);
}

static void
registry_handle_global_remove(void *data, struct wl_registry *registry,
		uint32_t name)
{
	// This space deliberately left blank
}

static const struct wl_registry_listener
registry_listener = {
	.global = registry_handle_global,
	.global_remove = registry_handle_global_remove,
};

int
main(int argc, char *argv[])
{
	struct wl_display *display = wl_display_connect(NULL);
	struct wl_registry *registry = wl_display_get_registry(display);
	wl_registry_add_listener(registry, &registry_listener, NULL);
	wl_display_roundtrip(display);
	return 0;
}
```

Feel free to reference previous chapters to interpret this program. We connect
to the display (chapter 4.1), obtain the registry (this chapter), add a listener
to it (chapter 3.4), then round-trip, handling the global event by printing the
globals available on this compositor. Try it for yourself:

请基于前面章节所学的内容来尽量理解这段代码。首先我们建立起与 display 之间的连接通道（第 4.1 章节的内容），
然后获取注册表对象（本章的内容），并为其添加一个监听器（第 3.4 章节的内容），最后使用 round-trip 来
处理 global event，即打印出该 compositor 上所有可用的 global 对象。你可以自己试试这个例子：

```c
$ cc -o globals -lwayland-client globals.c
```

**Note**: this chapter the last time we're going to show wire protocol dumps in
hexadecimal, and probably the last time you'll ever see them in general. A
better way to trace your Wayland client or server is to set the
`WAYLAND_DEBUG` variable in your environment to `1` before running your program.
Try it now with the "globals" program!

**注意**：这一章是我们最后一次展示十六进制的有线协议 dump，可能也是你最后一次看到它们的情况。
跟踪 Wayland client 或 server 的一个更好的方法是在运行程序之前将环境中的 WAYLAND_DEBUG 变量设置为 1。
现在试试 "globals" 程序吧!

原文链接:https://wayland-book.com/

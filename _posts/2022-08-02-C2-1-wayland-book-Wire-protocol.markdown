---
layout: post
title:  "2.1 Wayland book Protocol design:Wire protocol basics"
date:   2022-08-02 21:01:29 +0800
categories: jekyll update
---
# Wire protocol basics

**Note**: If you're just going to use libwayland, this chapter is optional -
feel free to skip to chapter 2.2.

**注意**：如果你只是使用 libwayland，本章是可选的 – 可跳到第 2.2 章。

---

The wire protocol is a stream of 32-bit values, encoded with the host's byte
order (e.g. little-endian on x86 family CPUs). These values represent the
following primitive types:

有线协议是一个 32-bit 值的流数据，用主机的字节顺序编码（例如 x86 系列 CPU 上的 little-endian）。
这些值基于以下原始类型：

**int, uint**: 32-bit signed or unsigned integer.

**fixed**: 24.8 bit signed fixed-point numbers.

**object**: 32-bit object ID.

**new_id**: 32-bit object ID which allocates that object when received.

**int, uint**: 32 位有符号或无符号整数。

**fixed**: 24.8 位有符号定点数(前面整数后面小数)。

**object**: 32 位对象 ID。

**new_id**: 32 位对象 ID。接收时分配该对象的 32 位对象 ID。

In addition to these primitives, the following other types are used:

除了这些外，还使用了以下其他类型： 

**string**: A string, prefixed with a 32-bit integer specifying its length (in
bytes), followed by the string contents and a NUL terminator, padded to 32
bits with undefined data. The encoding is not specified, but in practice UTF-8
is used.

**array**: A blob of arbitrary data, prefixed with a 32-bit integer specifying
its length (in bytes), then the verbatim contents of the array, padded to 32
bits with undefined data.

**fd**: 0-bit value on the primary transport, but transfers a file descriptor to
the other end using the ancillary data in the Unix domain socket message
(msg_control).

**enum**: A single value (or bitmap) from an enumeration of known constants,
encoded into a 32-bit integer.

**string**: 一个字符串，前 32 位整数指定其长度（以字节为单位），后跟字符串内容和
NUL 终止符，用未定义的数据填充不满 32 位部分。未指定编码，但实际上使用 UTF-8。

**array**: 前 32 位整数指定其长度（以字节为单位），然后是数组的内容，用未定义的数
据填充不满 32 位部分。。

**fd**: 优先传输的 0-bit 的值，使用 Unix 域套接字消息 (msg_control) 中的辅助数据将
文件描述符传输到另一端。

**enum**: 一个单独的值（或位图）用于枚举已知常量，编码为 32 位整数。

## Messages

The wire protocol is a stream of messages built with these primitives. Every
message is an event (in the case of server to client messages) or request
(client to server) which acts upon an *object*.

有线协议是用这些原语构建的消息流。每条消息都是作用于 *object* 的事件（传递方向：服务器到客户端）
或请求（传递方向：客户端到服务器）。

The message header is two words. The first word is the affected object ID. The
second is two 16-bit values; the upper 16 bits are the size of the message
(including the header itself) and the lower 16 bits are the event or request
opcode. The message arguments follow, based on a message signature agreed upon
in advance by both parties. The recipient looks up the object ID's interface and
the event or request defined by its opcode to determine the signature and nature
of the message.

消息头是两个字（32 位系统为 4 字节，64 位系统为 8 字节）。第一个字是将受到影响的 object ID。
第二个是两个 16-bit 的值；高 16 位是消息的大小（包括头本身），低 16 位是事件或请求的操作码。
消息参数基于双方事先商定的消息签名。接收者通过查找 object ID，找到对应的接口，定位由其操作码
定义的事件或请求，来确定消息的签名和性质。

To understand a message, the client and server have to establish the objects in
the first place. Object ID 1 is pre-allocated as the Wayland display singleton,
and can be used to bootstrap other objects. We'll discuss this in chapter 4. The
next chapter goes over what an interface is, and how requests and events work,
assuming you've already negotiated an object ID. Speaking of which...

要理解消息，客户端和服务器必须首先创建对象。Object ID 1 被预先分配给 Wayland display，
（wayland display）可用于引导其他对象。这一点我们将在第 4 章讨论。下一章将讨论什么是接口，
以及假设你已经协商了一个 object ID 的情况下请求和事件如何工作。

## Object IDs

When a message comes in with a `new_id` argument, the sender allocates an
object ID for it (the interface used for this object is established through
additional arguments, or agreed upon in advance for that request/event). This
object ID can be used in future messages, either as the first word of the
header, or as an `object_id` argument. The client allocates IDs in the range of
`[1, 0xFEFFFFFF]`, and the server allocates IDs in the range of `[0xFF000000,
0xFFFFFFFF]`. IDs begin at the lower end of this bound and increment with each
new object allocation.

当消息带有 `new_id` 参数时，发送方为其分配一个 object ID（用于此对象的接口或者是通过附加参数建立的，或者是预先商定的请求/事件）。
这个 object ID 可在未来的消息中用作第一个字或 `object_id` 参数。客户端分配 ID 的范围为 [1, 0xFEFFFFFF]，
服务器分配 ID 的范围为 [0xFF000000, 0xFFFFFFFF]。ID 从范围下限开始，并随着每个新对象分配而增加。

An object ID of 0 represents a null object; that is, a non-existent object or
the explicit lack of an object.

object ID 为 0 表示空对象；也就是说，一个不存在的对象或缺少显示申明的对象。

## Transports

To date all known Wayland implementations work over a Unix domain socket. This
is used for one reason in particular: file descriptor messages. Unix sockets are
the most practical transport capable of transferring file descriptors between
processes, and this is necessary for large data transfers (keymaps, pixel
buffers, and clipboard contents being the main use-cases). In theory, a
different transport (e.g. TCP) is possible, but someone would have to figure out
an alternative way of transferring bulk data.

迄今为止，所有已知的 Wayland 实现都在 Unix 域套接字上工作。这样做的主要原因是：文件描述符的传递依赖于 Socket。
Unix 套接字是最实用的能够在进程之间传输文件描述符的传输方式，这对于大数据传输（键映射、像素缓冲区和剪贴板内容是主要用例）
是必需的。理论上，不同的传输方式（例如 TCP）也是可以的，但必须有人想出一种可替换传输批量
数据的方式。

To find the Unix socket to connect to, most implementations just do what
libwayland does:

为了找到要连接的 Unix 套接字，大多数实现只需要通过 libwayland：

1. If `WAYLAND_SOCKET` is set, interpret it as a file descriptor number on which
   the connection is already established, assuming that the parent process
   configured the connection for us.
2. If `WAYLAND_DISPLAY` is set, concat with `XDG_RUNTIME_DIR` to form the path
   to the Unix socket.
3. Assume the socket name is `wayland-0` and concat with `XDG_RUNTIME_DIR` to
   form the path to the Unix socket.
4. Give up.

1.	如果 `WAYLAND_SOCKET` 已经设置，则将其解释为已经建立连接的文件描述符编号，
并假设父进程为我们配置了连接。
2.	如果 `WAYLAND_DISPLAY` 已设置，则将他与 `XDG_RUNTIME_DIR` 字符拼接，得到 Unix socket 的路径并连接。
3.	假设套接字名称是 `wayland-0`，与 `XDG_RUNTIME_DIR` 字符拼接，以形成 Unix socket 的路径并连接。
4.	放弃。


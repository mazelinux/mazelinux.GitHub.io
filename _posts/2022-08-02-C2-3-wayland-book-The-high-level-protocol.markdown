---
layout: post
title:  "2.3 Wayland book Protocol design:The high-level protocol"
date:   2022-08-02 21:08:29 +0800
categories: waylandbook
---
# The high-level protocol

In chapter 1.3, I mentioned that `wayland.xml` is probably installed with the
Wayland package on your system. Find and pull up that file now in your favorite
text editor. It's through this file, and others like it, that we define the
interfaces supported by a Wayland client or server.

在第 1.3 章中，我提到 `wayland.xml` 可能与 Wayland 软件包一起被安装。找到它，使用你
最喜欢的文本编辑器打开该文件。正是通过这个文件和其他类似的文件，我们定义了 Wayland 客户端或服务端支持的接口。

Each interface is defined in this file, along with its requests and events, and
their respective signatures. We use XML, everyone's favorite file format, for
this purpose. Let's look at the examples we discussed in the previous chapter
for `wl_surface`. Here's a sample:

这个文件中定义了接口，连同它的请求和事件，以及各自的签名。文件使用了每个人都
喜欢的 XML 格式。以上一章中讨论的 `wl_surface` 为例：

```xml
<interface name="wl_surface" version="4">
  <request name="damage">
    <arg name="x" type="int" />
    <arg name="y" type="int" />
    <arg name="width" type="int" />
    <arg name="height" type="int" />
  </request>

  <event name="enter">
    <arg name="output" type="object" interface="wl_output" />
  </event>
</interface>
```

**Note**: I've trimmed this snippet for brevity, but if you have the
`wayland.xml` file in front of you, seek out this interface and examine it
yourself &mdash; included is additional documentation explaining the purpose and
precise semantics of each request and event.

**注意**：为了简洁起见，我已经简化了代码，但如果你手头有 `wayland.xml` 这个文件，
可以寻找这个接口并自己查看它 &mdash; 包括附加文档，它解释了每个请求和事件的目的
和精确语义。

When processing this XML file, we assign each request and event an opcode in the
order that they appear (both numbered from zero and incrementing independently).
Combined with the list of arguments, you can decode the request or event when it
comes in over the wire, and based on the documentation shipped in the XML file
you can decide how to program your software to behave accordingly.  This usually
comes in the form of code generation &mdash; we'll talk about how libwayland 
does this in chapter 3.

在处理这个 XML 文件时，我们按照它们出现的顺序为每个请求和事件分配一个操作码（从零开始编号并独立递增）。
结合参数列表，你可以在请求或事件通过线路传入时对其进行解码，并且根据 XML 文件中提供的文档，
你可以自行决定如何对软件进行编程以实现相应的行为。这部分通常代码自动生成（wayland-scanner） &mdash; 我们将在第 3 章讨论 libwayland 如何做到这一点。

Starting from chapter 4, most of the remainder of this book is devoted to
explaining this file, as well as some supplementary protocol extensions.

从第 4 章开始，本书其余部分的大部分内容都致力于解释这个文件，以及一些补充协议扩展。

原文链接:https://wayland-book.com/

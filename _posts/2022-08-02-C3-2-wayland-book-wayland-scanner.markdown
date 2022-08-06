---
layout: post
title:  "3.2 Wayland book Libwayland in depth:Wayland-scanner"
date:   2022-08-02 21:19:29 +0800
categories: waylandbook
---
# wayland-scanner

The Wayland package comes with one binary: `wayland-scanner`. This tool is used
to generate C headers & glue code from the Wayland protocol XML files discussed
in chapter 2.3. This tool is used in the "wayland" package's build process to
pre-generate headers & glue code for the core protocol, `wayland.xml`. The
headers become `wayland-client-protocol.h` and `wayland-server-protocol.h` 
&mdash; though you normally include `wayland-client.h` and `wayland-server.h` 
instead of using these directly.

Wayland 软件包附带一个二进制工具: `wayland-scanner`。该工具用于从 Wayland 协议
文件（XML 文件，将在第 2.3 小节讨论）中生成 C 头文件和具体实现代码（又称粘合代码）。
在构建 "wayland" 包的过程中，该工具为 Wayland 核心协议 `wayland.xml`，提前生成头文件
和具体实现代码。通过 `wayland-scanner` 生成的头文件是 `wayland-client-protocol.h` 和
`wayland-server-protocol.h` &mdash; 但通常情况下，我们并不直接使用这两个头文件，而是
使用 `wayland-client.h` 和 `wayland-server.h` 代替。

The usage of this tool is fairly simple (and summarized by `wayland-scanner
-h`), but can be summed up as follows. To generate a client header:

这个工具的用法相当简单（可以通过 `wayland-scanner -h`查看它的具体使用方法），大致概括
为以下几点。

生成 client 端头文件:

    wayland-scanner client-header < protocol.xml > protocol.h

To generate a server header:

生成 server 端头文件:

    wayland-scanner server-header < protocol.xml > protocol.h

And to generate the glue code:

生成 client 和 server 端共同使用的协议实现代码（又称粘合代码）:

    wayland-scanner private-code < protocol.xml > protocol.c

Different build systems will have different approaches to configuring custom
commands &mdash; consult your build system's docs. Generally speaking, you'll
want to run `wayland-scanner` at build time, then compile and link your 
application to the glue code.

不同的构建系统将采用不同的方式来自定义构建命令(最终调用上面的那三个命令来生成协议文件)
 &mdash; 请查阅你本地构建系统的说明文档来获取这些命令。一般来说，你需要在构建时运行 `wayland-scanner`，
然后将粘合代码（`xxx-protocol.c`）和你的应用程序一起编译链接。

Go ahead and do this with any Wayland protocol now, if you have one handy
(`wayland.xml` is probably available in `/usr/share/wayland`, for example). Open
up the glue code & header and consult it as you read the following chapters, to
understand how the primitives offered by libwayland are applied in practice in
the generated code.

如果你手里有一份任意形式的 Wayland 协议，赶紧用上面的命令试一下吧！（举个例子，在 `/usr/share/wayland` 目录下
可能存放着一份 `wayland.xml` 文件）。请仔细阅读协议实现代码及其头文件，并在阅读下面的
章节时进行参考，从而进一步理解 libwayland 所提供的 primitive 在实际的粘合代码中是如何运用的。


原文链接:https://wayland-book.com/

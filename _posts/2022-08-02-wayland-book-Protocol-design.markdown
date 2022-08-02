---
layout: post
title:  "Wayland book Protocol design"
date:   2022-08-02 20:59:29 +0800
categories: jekyll update
---
# Protocol design

The Wayland protocol is built from several layers of abstraction. It starts with
a basic wire protocol format, which is a stream of messages decodable with
interfaces agreed upon in advance. Then we have higher level procedures for
enumerating interfaces, creating resources which conform to these interfaces,
and exchanging messages about them &mdash; the Wayland protocol and its 
extensions. On top of this we have some broader patterns which are frequently
used in Wayland protocol design. We'll cover all of these in this chapter.

Wayland 协议的构建基于多层抽象。最底层是拥有基本格式的有线协议，是事先定义好接口的可解码的消息流。
往上我们有更高级的程序枚举接口本身，创建这些接口的资源，并交换相关消息 &mdash; 基于 Wayland
以及 Wayland 扩展协议的消息。除此之外，我们还有一些更广泛的模式，这些模式在 Wayland 协议
设计中经常使用。我们将在本章中介绍所有这些协议。

Let's work our way from the bottom-up.

让我们从下到上地开始讲解。


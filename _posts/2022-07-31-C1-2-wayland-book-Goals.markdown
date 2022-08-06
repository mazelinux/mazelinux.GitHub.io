---
layout: post
title:  "1.2 Wayland book Introduction:Goals & target audience"
date:   2022-07-31 22:31:29 +0800
categories: jekyll update
---
# Goals & target audience

Our goal is for you to come away from this book with an understanding of the
Wayland protocol and its high-level usage. You should have a solid understanding
of everything in the core Wayland protocol, as well as the knowledge necessary
to evaluate and implement the various protocol extensions necessary for its
productive use. Primarily, this book uses the concerns of a Wayland client to
frame its presentation of Wayland. However, it should provide some utility for
those working on Wayland compositors as well.

我们的目标是让你在阅读完本书时，了解 Wayland 协议及其高级用法。对 Wayland 核心协议中
的所有内容有深入的了解，并（具备）评估和实施日常工作使用中，所需的各种协议扩展的知识。
最主要的，本书聚焦于 Wayland 客户端（的实现）来介绍 Wayland 的架构。不仅如此，它也为
那些在 Wayland compsitor 上工作的人提供一些实用的工具程序。

The free desktop ecosystem is complex and built from many discrete parts. We are
going to discuss these pieces very little &mdash; you won't find information 
here about leveraging libdrm in your Wayland compositor, or using libinput to 
process evdev events. Thus, this book is not a comprehensive guide for building 
Wayland compositors. We're also not going to talk about drawing technologies 
which are useful for Wayland clients, such as Cairo, Pango, GTK+, and so on, and 
thus neither is this a robust guide for the practical Wayland client 
implementation. Instead, we focus only on the particulars of Wayland.

免费桌面生态系统很复杂，由许多独立的部分组成。我们将很少讨论这些部分(除了 wayland 本身
的其他部分)；你不会在本书中找到有关在 Wayland compositor 中利用 libdrm，或使用 libinput
处理 evdev 事件的任何信息。因此，本书并不是构建 Wayland compositor 的综合性指南。我们
也不打算讨论对 Wayland 客户端来说（例如 Cairo、Pango、GTK+ 等）有用的绘图技术，因此本
书也不是实际的 Wayland 客户端实现的可靠指南。相对的，我们只关注 Wayland 本身的细节。

This book only covers the protocol and libwayland. If you are writing a client
and are already familiar with your favorite user interface rendering library,
bring your own pixels and we'll help you display them on Wayland. If you already
have an understanding of the technologies required to operate displays and input
devices for your compositor, this book will help you learn how to talk to
clients.

本书仅涵盖协议和 libwayland。如果你正在编写客户端并且已经（拥有）你最喜欢且熟悉的用户
界面渲染库，请带上你自己的像素渲染工具，我们将帮助你在 Wayland 上显示它们。如果你已经
了解为你的 compositor 操作显示器和输入设备所需的技术，本书将帮助你学习如何与客户端打交
道。

原文链接:https://wayland-book.com/

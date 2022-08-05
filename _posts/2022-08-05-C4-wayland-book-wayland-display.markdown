---
layout: post
title:  "4 Wayland book The Wayland display"
date:   2022-08-05 11:05:29 +0800
categories: jekyll update
---
# The Wayland display

Up to this point, we've left a crucial detail out of our explanation of how the
Wayland protocol manages joint ownership over objects between the client and
server: how those objects are created in the first place. The Wayland display,
or `wl_display`, implicitly exists on every Wayland connection. It has the
following interface:

就目前为止，我们在对 Wayland 协议如何管理 client 和 server 所共同拥有的 object 的讲解中，
还缺少一个关键的细节：这些 object 最初是如何创建的。（答案是）Wayland display，或者叫做 `w
l_display`。它隐式的存在于每一个（client 与 server 基于 socket 的） Wayland 连接上。
提供的接口如下：

```xml
<interface name="wl_display" version="1">
  <request name="sync">
    <arg name="callback" type="new_id" interface="wl_callback"
       summary="callback object for the sync request"/>
  </request>

  <request name="get_registry">
    <arg name="registry" type="new_id" interface="wl_registry"
      summary="global registry object"/>
  </request>

  <event name="error">
    <arg name="object_id" type="object" summary="object where the error occurred"/>
    <arg name="code" type="uint" summary="error code"/>
    <arg name="message" type="string" summary="error description"/>
  </event>

  <enum name="error">
    <entry name="invalid_object" value="0" />
    <entry name="invalid_method" value="1" />
    <entry name="no_memory" value="2" />
    <entry name="implementation" value="3" />
  </enum>

  <event name="delete_id">
    <arg name="id" type="uint" summary="deleted object ID"/>
  </event>
</interface>
```

The most interesting of these for the average Wayland user is `get_registry`,
which we'll talk about in detail in the following chapter. In short, the
registry is used to allocate other objects. The rest of the interface is used
for housekeeping on the connection, and are generally not important unless
you're writing your own libwayland replacement.

对于普遍的 Wayland 用户来说，其中最有趣的就是 `get_registry`，我们将在下一章讨论它的细节。
简单说来，registry 主要是用来分配其他 object 的。除此之外的剩余部分接口则用于对连接的内部管理，
不过除非你正在编写自己的 libwayland，否则，剩余部分通常并不重要。

Instead, this chapter will focus on a number of functions that libwayland
associated with the `wl_display` object, for establishing and maintaining your
Wayland connection. These are used to manipulate libwayland's internal state,
rather than being directly related to wire protocol requests and events.

本章将重点介绍 libwayland 中如何建立和维护 Wayland 连接, 与 `wl_display` object 关联的一些函数。
他们仅仅用于设置 libwayland 的内部状态，不与协议中的 request 和 event 直接相关。 

We'll start with the most important of these functions: establishing the
display. For clients, this will cover the actual process of connecting to the
server, and for servers, the process of configuring a display for clients to
connect to.

我们将从这些功能中最重要的部分开始介绍：建立 display。 对于 client，（建立 display）将包含连接到
server 的实际操作过程，而对于 server，将包含为 client 创建并配置用于连接的 display 的全部过程。

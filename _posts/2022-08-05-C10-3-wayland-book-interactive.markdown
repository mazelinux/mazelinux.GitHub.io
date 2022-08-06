---
layout: post
title:  "10.3 Wayland book Interactive move and resize"
date:   2022-08-05 15:16:29 +0800
categories: waylandbook
---
# Interactive move and resize

Many application windows have interactive UI elements the user can use to drag
around or resize windows. Many Wayland clients, by default, expect to be
responsible for their own window decorations to provide these interactive
elements. On X11, application windows could position themselves independently
anywhere on the screen, and used this to facilitate these interactions.

许多应用程序窗口都有交互式 UI 元素，用户可以使用它来拖动窗口或调整窗口大小。 默认情况下，
许多 Wayland 客户希望负责自己的窗口装饰以提供这些交互元素。 在 X11 上，应用程序窗口可以将
自己独立地定位在屏幕上的任何位置，并使用它来促进这些交互。

However, a deliberate design trait of Wayland makes application windows ignorant
of their exact placement on screen or relative to other windows. This decision
affords Wayland compositors a greater deal of flexibility &mdash; windows could 
be shown in several places at once, arranged in the 3D space of a VR scene, or
presented in any other novel way. Wayland is designed to be generic and widely
applicable to many devices and form factors.

然而，Wayland 的一个深思熟虑的设计特征使得应用程序窗口不知道它们在屏幕上或相对于其他窗口
的确切位置。 这一决定为 Wayland 合成师提供了更大的灵活性——窗口可以同时在多个地方显示，
在 VR 场景的 3D 空间中排列，或者以任何其他新颖的方式呈现。 Wayland 被设计为通用且广泛适用
于许多设备和外形。

To balance these two design needs, XDG toplevels offer two requests which can be
used to ask the compositor to begin an interactive move or resize operation. The
relevant parts of the interface are:

为了平衡这两种设计需求，XDG 顶层提供了两个请求，可用于要求合成器开始交互式移动或
调整大小操作。 界面的相关部分是：

```xml
<request name="move">
  <arg name="seat" type="object" interface="wl_seat" />
  <arg name="serial" type="uint" />
</request>
```

Like the popup creation request explained in the previous chapter, you have to
provide an input event serial to start an interactive operation. For example,
when you receive a mouse button down event, you can use that event's serial to
begin an interactive move operation. The compositor will take over from here,
and begin an interactive operation to your window in its internal coordinate
space.

就像上一章解释的弹出创建请求一样，你必须提供一个输入事件序列来启动交互操作。 例如，
当您收到鼠标按钮按下事件时，您可以使用该事件的序列号来开始交互式移动操作。 合成器将
从这里接管，并在其内部坐标空间中开始对您的窗口进行交互操作。

Resizing is a bit more complex, due to the need to specify which edges or
corners of the window are implicated in the operation:

调整大小有点复杂，因为需要指定窗口的哪些边缘或角参与操作：

```xml
<enum name="resize_edge">
  <entry name="none" value="0"/>
  <entry name="top" value="1"/>
  <entry name="bottom" value="2"/>
  <entry name="left" value="4"/>
  <entry name="top_left" value="5"/>
  <entry name="bottom_left" value="6"/>
  <entry name="right" value="8"/>
  <entry name="top_right" value="9"/>
  <entry name="bottom_right" value="10"/>
</enum>

<request name="resize">
  <arg name="seat" type="object" interface="wl_seat" />
  <arg name="serial" type="uint" />
  <arg name="edges" type="uint" />
</request>
```

But otherwise, it functions much the same. If the user clicks and drags along
the bottom-left corner of your window, you may want to send an interactive
resize request with the corresponding seat & serial, and set the edges argument
to bottom_left.

但除此之外，它的功能大致相同。 如果用户在窗口的左下角单击并拖动，您可能需要发送一个
带有相应座位和序列号的交互式调整大小请求，并将edges 参数设置为bottom_left。

There's one additional request necessary for clients to totally implement
interactive client-side window decorations:

客户端完全实现交互式客户端窗口装饰还需要一个额外的请求：

```xml
<request name="show_window_menu">
  <arg name="seat" type="object" interface="wl_seat" />
  <arg name="serial" type="uint" />
  <arg name="x" type="int" />
  <arg name="y" type="int" />
</request>
```

A contextual menu offering window operations, such as closing or minimizing the
window, is often raised when clicking on window decorations. For clients where
window decorations are managed by the client, this serves to link the
client-driven interactions with compositor-driven meta operations like
minimizing windows. If your client uses client-side decorations, you may use
this request for this purpose.

单击窗口装饰时，通常会弹出提供窗口操作（例如关闭或最小化窗口）的上下文菜单。 对于
窗口装饰由客户端管理的客户端，这用于将客户端驱动的交互与合成器驱动的元操作（如最小化窗口）
联系起来。 如果您的客户使用客户端装饰，您可以为此目的使用此请求。

## xdg-decoration

The last detail which bears mentioning when discussing the behavior of
client-side decorations is the protocol which governs the negotiation of their
use in the first place. Different Wayland clients and servers may have different
preferences about the use of server-side or client-side window decorations. To
express these intentions, a protocol extension is used: `xdg-decoration`. It can
be found in wayland-protocols. The protocol provides a global:

在讨论客户端装饰的行为时，最后一个值得一提的细节是首先管理其使用协商的协议。 不同的
Wayland 客户端和服务器可能对使用服务器端或客户端窗口装饰有不同的偏好。 为了表达这些意图，
使用了协议扩展：xdg-decoration。 它可以在wayland-protocols 中找到。 该协议提供了一个全局的：

```xml
<interface name="zxdg_decoration_manager_v1" version="1">
  <request name="destroy" type="destructor" />

  <request name="get_toplevel_decoration">
    <arg name="id" type="new_id" interface="zxdg_toplevel_decoration_v1"/>
    <arg name="toplevel" type="object" interface="xdg_toplevel"/>
  </request>
</interface>
```

You may pass your xdg_toplevel object into the `get_toplevel_decoration` request
to obtain an object with the following interface:

您可以将 xdg_toplevel 对象传递到 get_toplevel_decoration 请求中以获取具有以下接口的对象：

```xml
<interface name="zxdg_toplevel_decoration_v1" version="1">
  <request name="destroy" type="destructor" />

  <enum name="mode">
    <entry name="client_side" value="1" />
    <entry name="server_side" value="2" />
  </enum>

  <request name="set_mode">
    <arg name="mode" type="uint" enum="mode" />
  </request>

  <request name="unset_mode" />

  <event name="configure">
    <arg name="mode" type="uint" enum="mode" />
  </event>
</interface>
```

The `set_mode` request is used to express a preference from the client, and
`unset_mode` is used to express no preference. The compositor will then use the
`configure` event to tell the client whether or not to use client-side
decorations. For more details, consult the full XML.

set_mode 请求用于表示来自客户端的偏好，而 unset_mode 用于表示没有偏好。 然后合成器
将使用配置事件告诉客户端是否使用客户端装饰。 有关更多详细信息，请参阅完整的 XML。

原文链接:https://wayland-book.com/

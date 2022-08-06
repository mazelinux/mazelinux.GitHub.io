---
layout: post
title:  "10.2 Wayland book Popups & parent windows"
date:   2022-08-05 15:15:29 +0800
categories: waylandbook
---
# Popups

When designing software which utilizes application windows, there are many cases
where smaller secondary surfaces are used for various purposes. Some examples
include context menus which appear on right click, dropdown boxes to select a
value from several options, contextual hints which are shown when you hover the
mouse over a UI element, or menus and toolbars along the top and bottom of a
window. Often these will be nested, for example, by following a path like "File
→ Recent Documents → Example.odt".

在设计利用应用程序窗口的软件时，很多情况下会出于各种目的使用较小的次表面。 一些示例
包括右键单击时出现的上下文菜单、从多个选项中选择值的下拉框、将鼠标悬停在 UI 元素上时
显示的上下文提示，或窗口顶部和底部的菜单和工具栏。 通常这些会被嵌套，例如，通过遵循类似
“文件 → 最近的文档 → Example.odt”的路径。

For Wayland, the XDG shell provides facilities for managing these windows:
popups. We looked at `xdg_surface`'s "`get_toplevel`" request for creating
top-level application windows earlier. In the case of popups, the "`get_popup`"
request is used instead.

对于 Wayland，XDG shell 提供了管理这些窗口的工具：弹出窗口。 我们之前查看了 xdg_surface
用于创建顶级应用程序窗口的“get_toplevel”请求。 在弹出窗口的情况下，使用“get_popup”请求代替。

```xml
<request name="get_popup">
  <arg name="id" type="new_id" interface="xdg_popup"/>
  <arg name="parent" type="object" interface="xdg_surface" allow-null="true"/>
  <arg name="positioner" type="object" interface="xdg_positioner"/>
</request>
```

The first and second arguments are reasonably self-explanatory, but the third
one introduces a new concept: positioners. The purpose of the positioner is, as
the name might suggest, to *position* the new popup. This is used to allow the
compositor to participate in the positioning of popups using its priveleged
information, for example to avoid having the popup extend past the edge of the
display. We'll discuss positioners in chapter 10.4, for now you can simply create
one and pass it in without further configuration to achieve reasonably sane
default behavior, utilizing the appropriate `xdg_wm_base` request:

第一个和第二个参数不言自明，但第三个参数引入了一个新概念：定位器。 顾名思义，定位器
的目的是定位新的弹出窗口。 这用于允许合成器使用其特权信息参与弹出窗口的定位，例如避免
弹出窗口延伸超过显示的边缘。 我们将在第 10.4 章讨论定位器，现在您可以简单地创建一个并
将其传递进来，无需进一步配置，以利用适当的 xdg_wm_base 请求实现合理合理的默认行为：

```xml
<request name="create_positioner">
  <arg name="id" type="new_id" interface="xdg_positioner"/>
</request>
```

So, in short, we can:

所以，简而言之，我们可以：

1. Create a new `wl_surface`
2. Obtain an `xdg_surface` for it
3. Create a new `xdg_positioner`, saving its configuration for chapter 10.4
4. Create an `xdg_popup` from our XDG surface and XDG positioner, assigning its
   parent to the `xdg_toplevel` we created earlier.
   
1. 创建一个新的`wl_surface`
2. 为它获取一个`xdg_surface`
3. 新建一个`xdg_positioner`，保存10.4章节的配置
4. 从我们的 XDG 表面和 XDG 定位器创建一个 `xdg_popup`，将其父级分配给我们之前创建的 `xdg_toplevel`。

Then we can render and attach buffers to our popup surface with the same
lifecyle discussed earlier. We also have access to a few other popup-specific
features.

然后我们可以使用前面讨论的相同生命周期将缓冲区渲染并附加到我们的弹出表面。 我们
还可以访问其他一些特定于弹出窗口的功能。

## Configuration

Like the XDG toplevel configure event, the compositor has an event which it may
use to suggest the size for your popup to assume. Unlike toplevels, however,
this also includes a positioning event, which informs the client as to the
position of the popup relative to its parent surface.

像 XDG 顶级配置事件一样，合成器有一个事件，它可以用来建议弹出窗口的大小。 然而，与
顶层不同的是，这还包括一个定位事件，它通知客户端弹出窗口相对于其父表面的位置。

```xml
<event name="configure">
  <arg name="x" type="int"
 summary="x position relative to parent surface window geometry"/>
  <arg name="y" type="int"
 summary="y position relative to parent surface window geometry"/>
  <arg name="width" type="int" summary="window geometry width"/>
  <arg name="height" type="int" summary="window geometry height"/>
</event>
```

The client can influence these values with the XDG positioner, to be discussed
in chapter 10.4.

客户可以使用 XDG 定位器影响这些值，这将在第 10.4 章中讨论。

## Popup grabs

Popup surfaces will often want to "grab" all input, for example to allow the
user to use the arrow keys to select different menu items. This is facilitiated
through the grab request:

弹出窗口通常希望“抓取”所有输入，例如允许用户使用箭头键选择不同的菜单项。这是通过抓取
请求促进的：

```xml
<request name="grab">
  <arg name="seat" type="object" interface="wl_seat" />
  <arg name="serial" type="uint" />
</request>
```

A prerequisite of this request is having received a qualifying input event, such
as a right click. The serial from this input event should be used in this
request. These semantics are covered in detail in chapter 9. The compositor can
cancel this grab later, for example if the user presses escape or clicks outside
of your popup.

此请求的先决条件是已收到合格的输入事件，例如右键单击。 此输入事件中的序列应在此请求中使用。
第 9 章详细介绍了这些语义。合成器可以稍后取消此抓取，例如，如果用户按下退出键或在弹出窗口
之外单击。

## Dismissal

In these cases where the compositor dismisses your popup, such as when the
escape key is pressed, the following event is sent:

在合成器关闭弹出窗口的这些情况下，例如按下转义键时，将发送以下事件：

```xml
<event name="popup_done" />
```

To avoid race conditions, the compositor keeps the popup structures in memory
and services requests for them even after their dismissal. For more detail about
object lifetimes and race conditions, see chapter 2.4.

为了避免竞争条件，合成器将弹出结构保留在内存中，即使在它们被解除后也为它们提供服务请求。
有关对象生命周期和竞争条件的更多详细信息，请参阅第 2.4 章。

## Destroying popups

Client-initiated destruction of a popup is fairly straightforward:

客户端启动的弹出窗口销毁相当简单：

```xml
<request name="destroy" type="destructor" />
```

However, one detail bears mentioning: you must destroy all popups from the
top-down. The only popup you can destroy at any given moment is the top-most
one. If you don't, you'll be disconnected with a protocol error.

但是，有一个细节值得一提：您必须自上而下销毁所有弹出窗口。 您可以在任何特定时刻销毁的唯
一弹出窗口是最顶部的弹出窗口。 如果不这样做，您将因协议错误而断开连接。

原文链接:https://wayland-book.com/

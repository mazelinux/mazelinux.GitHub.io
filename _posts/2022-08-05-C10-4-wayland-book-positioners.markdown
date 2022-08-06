---
layout: post
title:  "10.4 Wayland book Positioners"
date:   2022-08-05 15:17:29 +0800
categories: waylandbook
---
# Positioners

When we introduced pop-ups a few pages ago, we noted that you had to provide a
positioner object when creating the pop-up. We asked you not to worry about it
and just use the defaults, because it's a complicated interface and was beside
the point. Now, we'll explore this complex interface in depth.

当我们在几页前介绍弹出窗口时，我们注意到您必须在创建弹出窗口时提供定位器对象。 我们
要求您不要担心它，只需使用默认值，因为它是一个复杂的界面，而且无关紧要。 现在，我们
将深入探索这个复杂的界面。

When you open a pop-up window, it's shown in a windowing system which has
constraints that your client is not privy to. For example, Wayland clients are
unaware of where their windows are being shown on-screen. Therefore, if you
right click a window, the client does not poses the necessary information to
determine that the resulting pop-up might end up running itself off the edge of
the screen. The positioner is designed to address these issues, by letting the
client specify certain constraints in how the pop-up can be moved or resized,
and then the compositor, being in full possession of the facts, can make the
final call on how to accommodate.

当您打开一个弹出窗口时，它会显示在一个窗口系统中，该窗口系统具有您的客户不知道的限制条件。
例如，Wayland 客户不知道他们的窗口显示在屏幕上的哪个位置。 因此，如果您右键单击一个窗口，
客户端不会提供必要的信息来确定生成的弹出窗口最终可能会在屏幕边缘运行。 定位器旨在解决这些
问题，通过让客户端指定如何移动或调整弹出窗口大小的某些约束，然后合成器在完全掌握事实的情况
下可以对如何适应 .

# The Basics

```xml
<request name="destroy" type="destructor"></request>
```

This destroys the positioner when you're done with it. You can call this after
your pop-up has been created.

完成后，这会破坏定位器。 您可以在创建弹出窗口后调用它。

```xml
<request name="set_size">
  <arg name="width" type="int" />
  <arg name="height" type="int" />
</request>
```

The set_size request is used to set the size of the pop-up window being created.

set_size 请求用于设置正在创建的弹出窗口的大小。

All clients which use a positioner will use these two requests. Now, let's get
to the interesting ones.

所有使用定位器的客户端都将使用这两个请求。 现在，让我们来看看有趣的。

# Anchoring

```xml
```

原文链接:https://wayland-book.com/

---
layout: post
title:  "1.1 Wayland book Introduction:High-level design"
date:   2022-07-31 22:31:29 +0800
categories: jekyll update
---
# High-level design

Your computer has *input* and *output* devices, which respectively are
responsible for receiving information from you and displaying information to
you. These input devices take the form of, for example:

你的计算机有 *输入* 和 *输出* 设备，分别负责接收来自用户的的信息和向用户显示信息。
例如下面这些输入设备：

- Keyboards
- Mice
- Touchpads
- Touch screens
- Drawing tablets

- 键盘
- 鼠标
- 触摸板
- 触摸屏
- 电脑绘图板

Your output devices generally take the form of displays, on your desk or your
laptop or mobile device. These resources are shared between all of your
applications, and the role of the **Wayland compositor** is to dispatch input
events to the appropriate **Wayland client** and to display their windows in
their appropriate place on your outputs. The process of bringing together all of
your application windows for display on an output is called *compositing* 
&mdash; and thus we call the software which does this the *compositor*.

你的输出设备通常以显示器的形式出现，在你的台式机上、笔记本电脑上或移动设备上。这些
资源（输入输出设备）在所有应用程序之间共享，**Wayland compositor** 的作用是将输入事件
派发给相应的 **Wayland client**，并在输出设备的对应位置显示它们的窗口。把你所有的应
用程序窗口放在一起（堆叠在一个屏幕上）以在输出设备上显示的过程叫做 *compositing* &mdash;
因此我们把做这个的软件叫做 *compositor*。

## In practice

There are many distinct software components in desktop ecosystem. There are
tools like Mesa for rendering (and each of its drivers), the Linux KMS/DRM
subsystem, buffer allocation with GBM, the userspace libdrm library, libinput
and evdev, and much more still. Don't worry &mdash; expertise with most of these
systems is not required for understanding Wayland, and in any case are largely
beyond the scope of this book. In fact, the Wayland protocol is quite
conservative and abstract, and a Wayland-based desktop could easily be built &
run most applications without implicating any of this software. That being said,
a surface-level understanding of what these pieces are and how they work is
useful. Let's start from the bottom and work our way up.

桌面生态系统中有许多不同的软件组件。有像 Mesa 这样的渲染工具（及其每个驱动程序）、
Linux KMS/DRM 子系统、缓冲区分配 GBM、用户空间库 libdrm、libinput 和 evdev 等等。别
担心 &mdash; 要理解 Wayland ，并不需要具备这些系统的专业知识，这在很大程度上超出了本
书的范围。事实上，Wayland 协议相当保守和抽象，基于 Wayland 的桌面可以轻松构建和运行
大多数应用程序，且不会涉及任何此类软件。话虽如此，大概理解这些部分是什么，如何工作
的还是很有用的。让我们从底层开始，逐步向上介绍。

## The hardware

A typical computer is equipped with a few important pieces of hardware. Outside
of the box, we have your displays, keyboard, mouse, perhaps some speakers and a
cute USB cup warmer. There are several components *inside* the box for
interfacing with these devices. Your keyboard and mouse, for example, are
probably plugged into USB ports, for which your system has a dedicated USB
controller. Your displays are plugged into your GPU.

一台典型的计算机配备许多重要的硬件。主机外，有显示器、键盘、鼠标，也许还有扬声器和一
个可爱的 USB 接口保温杯。主机里有几个组件与这些（外围）设备连接。例如，你的键盘和鼠标
可能是接入 USB 端口的，系统为此具有专用的 USB 控制器。而显示器则与 GPU 相连。

These systems have their own jobs and state. For example, your GPU has state
in the form of memory for storing pixel buffers in, and jobs like *scanning
out* these buffers to your displays. Your GPU also provides a processor which is
specially tuned to be good at highly parallel jobs (such as calculating the
right color for each of the 2,073,600 pixels on a 1080p display), and bad at
everything else. The USB controller has the unenviable job of implementing the
legendarily dry USB specification for receiving input events from your keyboard,
or instructing your coaster to assume a temperature carefully selected to at
once avoid lawsuits and frustrate you with cold coffee.

这些子系统有自己（独立）的工作和状态。例如，状态方面，GPU 有存储像素缓冲区的内存（台式机一般为自己的显存），
工作方面，GPU 扫描这些缓冲区，呈现到显示器。GPU 提供一个专门特别优化过的处理器，它擅长
高度并行的工作(比如在 1080p 的显示器上为 2073600 个像素计算正确的颜色)，而在其他方面表
现不佳[是指和 CPU 比较，如逻辑判断方面]。USB 控制器有一项令人羡慕的工作，即实现传说中
的枯燥的 USB 规范，以接收来自键盘的输入事件，或指示你的杯垫使用精心选择的温度，以避免
（不符合 USB 规范）诉讼和冷咖啡寒了你的心。

At this level, your hardware has little concept of what applications are running
on your system. The hardware provides an interface with which it can be
commanded to perform work, and does what it's told &mdash; regardless of who 
tells it so. For this reason, only one component is allowed to talk to it...

在这个层面上，硬件对系统上正在运行的应用程序几乎没有概念。硬件提供接口，（通过接口）可
以命令它执行工作，并按照指示去做——不管是谁告诉它的。出于这个原因，（同一时刻）只允许一
个组件与它通信......

## The kernel

This responsibility falls onto the kernel. The kernel is a complex beast, so
we'll focus on only the parts which are relevant to Wayland. Linux's job is to
provide an abstraction over your hardware, so that they can be safely accessed
by *userspace* &mdash; where our Wayland compositors run. For graphics, this is
called **DRM**, or *direct rendering manager*, for efficiently tasking the GPU
with work from userspace. An important subsystem of DRM is **KMS**, or *kernel
mode setting*, for enumerating your displays and setting properties such as 
their selected resolution (also known as their "mode"). Input devices are 
abstracted through an interface called **evdev**.

（接下来的）责任落在了内核上。内核是超级复杂庞大的，所以我们将只关注与 Wayland 相关
的部分。Linux 的工作是为硬件提供抽象，以便它们可以被 *用户空间* 安全地访问 &mdash; 我们
的 Wayland compositor 就在那里运行。对于图形框架来说，这块叫做 **DRM**，或 *direct rendering manager*，
用于有效地将用户空间的工作分配给 GPU。DRM 的一个重要子系统是 **KMS**，或 *kernel mode setting*，
用于枚举不同的显示器和设置相关的属性，例如，选择（屏幕）分辨率（也称为它们的 "mode"）。
输入设备是通过一个名为 **evdev** 的接口抽象出来的。

Most kernel interfaces are made available to userspace by way of special files
in `/dev`. In the case of DRM, these files are in `/dev/dri/`, usually in the
form of a primary node (e.g. `card0`) for privileged operations like
modesetting, and a render node (e.g. `renderD128`), for unprivileged operations
like rendering or video decoding. For evdev, the "device nodes" are
`/dev/input/event*`.

大多数内核接口都通过在 `/dev` 中的特殊文件，从而暴露给用户空间。在 DRM 这边，这些
文件在 `/dev/dri/` 中，通常包括用于特权操作如模式设置的主节点（例如`card0`），和用于
非特权操作如渲染或视频解码渲染节点（例如`renderD128`）。对于 evdev，"设备节点" 是
`/dev/input/event*`。

## Userspace

Now, we enter userspace. Here, applications are isolated from the hardware and
must work via the device nodes provided by the kernel.

现在我们进入用户空间。在这儿，应用程序想要与硬件（打交道），必须通过内核提供的设备节点。

### libdrm

Most Linux interfaces have a userspace counterpart which provides a
pleasant(ish) C API for working with these device nodes. One such library is
libdrm, which is the userspace portion of the DRM subsystem. libdrm is used by
Wayland compositors to do modesetting and other DRM operations, but is generally
not used by Wayland clients directly.

大多数 Linux 接口提供方便的基于 C API 的用户空间的对应实现，这些实现直接操纵设备节点。
Libdrm 就是其中之一，是 DRM 子系统中用户空间的那部分。Wayland 合成器使用 libdrm 完成 modesetting 
和一些其他的 DRM 相关操作，但 Wayland 客户端通常不会直接调用（libdrm）。

### Mesa

Mesa is one of the most important parts of the Linux graphics stack. It
provides, among other things, vendor-optimized implementations of OpenGL (and
Vulkan) for Linux and the **GBM** (Generic Buffer Management) library &mdash; an
abstraction on top of libdrm for allocating buffers on the GPU. Most Wayland
compositors will use both GBM and OpenGL via Mesa, and most Wayland clients will
use at least its OpenGL or Vulkan implementations.

Mesa 是 Linux 图形栈中最重要的组成部分之一。它为 Linux 提供了供应商特别优化的 OpenGL（还有Vulkan）
实现和 **GBM**（Generic Buffer Management）库 &mdash; 一个在 libdrm 之上的库，用于在
GPU 上申请缓冲区。大多数 Wayland 合成器会使用 Mesa 里面的 GBM 和 OpenGL，而大多数 Wayland
客户端至少会使用它（Mesa）的 OpenGL 或者 Vulkan 实现（客户端不一定通过 GBM 去申请 buffer）。

### libinput

Like libdrm abstracts the DRM subsystem, libinput provides the userspace end of
evdev. It's responsible for receiving input events from the kernel from your
various input devices, decoding them into a usable form, and passing them on to
the Wayland compositor. The Wayland compositor requires special permissions to
use the evdev files, forcing Wayland clients to go through the compositor to
receive input events &mdash; which, for example, prevents keylogging.

就像 libdrm 抽象了 DRM 子系统一样，libinput 提供了用户空间部分的 evdev。 它负责从内核
接收来自各种输入设备的输入事件，将它们解码成（Wayland 合成器）可用的形式，并将它们传递
给 Wayland 合成器。Wayland 合成器需要特殊权限才能访问 evdev 文件节点，而 Wayland 客户
端只能通过合成器接收输入事件 &mdash；例如，它以防止键盘事件被记录（如果 client 能够直接
监听 input 事件，在你输入密码时，事件也会上报到对应节点，这很危险）。

### (e)udev

Dealing with the appearance of new devices from the kernel, configuring
permissions for the resulting device nodes in `/dev`, and sending word of these
changes to applications running on your system, is a responsibility that falls
onto userspace. Most systems use udev (or eudev, a fork) for this purpose. Your
Wayland compositor uses udev to enumerate input devices and GPUs, and to receive
notifications when new ones appear or old ones are unplugged.

在处理在内核中新出现的设备时（热插拔事件），需要在 /dev 中生成设备节点并配置权限，将这
些更改的消息发送到系统上运行的应用程序，上述操作都是用户空间的任务。大多数系统使用 udev（或 eudev，一个分支）
完成此任务。Wayland 合成器使用 udev 来枚举（热插拔的）输入设备和 GPU，并在出现新设备和
（移除）旧设备时接收通知。

### xkbcommon

XKB, short for X keyboard, is the original keyboard handling subsystem of the
Xorg server. Several years ago, it was extracted from the Xorg tree and made
into an independent library for keyboard handling, and it no longer has any
practical relationship with X. Libinput (along with the Wayland compositor)
delivers keyboard events in the form of scancodes, whose precise meaning varies
from keyboard to keyboard. It's the responsibility of xkbcommon to translate
these scan codes into meaningful and generic key "symbols" &mdash; for example,
converting `65` to `XKB_KEY_Space`. It also contains a state machine which knows
that pressing "1" while shift is held emits "!".

XKB 是 X keyboard 的缩写，是 Xorg 服务器的原始键盘处理子系统。几年前，它被从 Xorg 中提
取出来，做成一个独立的键盘处理库，与 X 不再有任何实际关系。Libinput（连同 Wayland 合成器）
以扫描码的形式提供键盘事件，其准确的含义因键盘而异。xkbcommon 负责将这些扫描代码转换为
有意义的通用键 "符号" &mdash; 例如，将 `65` 转换为 `XKB_KEY_Space`。它还包含一个状态机，
它知道在按住 `shift` 时按 "1" 会发出 "!"。

### pixman

A simple library used by clients and compositors alike for efficiently
manipulating pixel buffers, doing math with intersecting rectangles, and
performing other similar **pix**el **man**ipulation tasks.

客户端和合成器使用的一个简单库，用于有效操作像素缓冲区、使用相交矩形进行数学运算以及执
行其他类似的像素操作任务。

### libwayland

libwayland the most commonly used implementation of the Wayland protocol,
is written in C, and handles much of the low-level wire protocol. It also
provides a tool which generates high-level code from Wayland protocol
definitions (which are XML files). We will be discussing libwayland in detail in
chapter 1.3, and throughout this book.

libwayland 是 Wayland 协议最常用的实现，是用 C 编写的，可以处理许多底层的有线协议。 它
还提供了一个工具，可以从 Wayland 协议定义（XML 文件）生成高级代码。 我们将在本书的第 1.3
章中详细讨论 libwayland。

### ...and all the rest.

Each of the pieces mentioned so far are consistently found throughout the Linux
desktop ecosystem. Beyond this, more components exist. Many graphical
applications don't know about Wayland at all, choosing instead to allow
libraries like GTK+, Qt, SDL, and GLFW &mdash; among many others &mdash; to deal 
with it. Many compositors choose software like wlroots to abstract more of their
responsibilities, while others implement everything in-house.

到目前为止提到的每个部分都能在整个 Linux 桌面生态系统中找到。 除此之外，还存在更多组件。
许多图形应用程序根本不了解 Wayland，而是选择 GTK+、Qt、SDL 和 GLFW 等库来处理它。 许多
合成器处理者选择 wlroots 之类的软件来抽象出他们（合成器）的更多职责，而另一些人则在（合成器）
内部实现一切。

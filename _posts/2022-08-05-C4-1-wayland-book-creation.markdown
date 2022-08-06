---
layout: post
title:  "4.1 Wayland book Creating a display"
date:   2022-08-05 11:07:29 +0800
categories: jekyll update
---
# Creating a display

Fire up your text editor &mdash; it's time to write our first lines of code.

启动你的文本编辑器 &mdash; 是时候编写我们的第一行代码了。

## For Wayland clients

Connecting to a Wayland server and creating a `wl_display` to manage the
connection's state is quite easy:

连接到 Wayland server 并创建一个 `wl_display` 来管理连接状态，是非常简单的：

```c
#include <stdio.h>
#include <wayland-client.h>

int
main(int argc, char *argv[])
{
    struct wl_display *display = wl_display_connect(NULL);
    if (!display) {
        fprintf(stderr, "Failed to connect to Wayland display.\n");
        return 1;
    }
    fprintf(stderr, "Connection established!\n");

    wl_display_disconnect(display);
    return 0;
}
```

Let's compile and run this program. Assuming you're using a Wayland compositor
as you read this, the result should look like this:

如果你在阅读本文时使用的是 Wayland compositor，编译并运行这段代码后，将得到如下结果：

```sh
$ cc -o client client.c -lwayland-client
$ ./client
Connection established!
```

`wl_display_connect` is the most common way for clients to establish a Wayland
connection. The signature is:

`wl_display_connect` 是 client 用于建立 Wayland 连接最常用的一种方式。声明如下：

```c
struct wl_display *wl_display_connect(const char *name);
```

The "name" argument is the name of the Wayland display, which is typically
`"wayland-0"`. You can swap the `NULL` for this in our test client and try for
yourself &mdash; it's likely to work. This corresponds to the name of a Unix 
socket in `$XDG_RUNTIME_DIR`. `NULL` is preferred, however, in which case 
libwayland will:

"name" 参数是 Wayland display 的名字，通常是 `"wayland-0"`（也就是 socket 的名字，client 
通过连接此 socket 地址，而与 sever 交互）。 你可以尝试在我们的测试 client 中将它设置
为 `NULL` &mdash; 它大概率也正常工作。这取决于 `$XDG_RUNTIME_DIR` 中的
Unix 套接字的名称。在多数情况下，`NULL` 是可行的，因为 libwayland 将： 

1. If `$WAYLAND_DISPLAY` is set, attempt to connect to
   `$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY`
2. Attempt to connect to `$XDG_RUNTIME_DIR/wayland-0`
3. Fail :(

1.如果设置了 `$WAYLAND_DISPLAY`，尝试连接到 `$XDG_RUNTIME_DIR/$WAYLAND_DISPLAY`
2.否则，尝试连接到 `$XDG_RUNTIME_DIR/wayland-0`
3.不然，失败:( 

This allows users to specify the Wayland display they want to run their clients
on by setting the `$WAYLAND_DISPLAY` to the desired display. If you have more
complex requirements, you can also establish the connection yourself and create
a Wayland display from a file descriptor:

用户可以通过 `$WAYLAND_DISPLAY` 设置特定的 display socket 地址，从而指定特定的 Wayland display
同 client 建立连接。如果你有更复杂的需求，你也可以从文件描述符中创建一个 Wayland display，
从而建立自己的连接：

```c
struct wl_display *wl_display_connect_to_fd(int fd);
```

You can also obtain the file descriptor that the `wl_display` is using via
`wl_display_get_fd`, regardless of how you created the display.

你还可以通过 `wl_display_get_fd` 获取 `wl_display` 正在使用的文件描述符，而无需关注
这个 display 是如何创建的。 

```c
int wl_display_get_fd(struct wl_display *display);
```

## For Wayland servers

The process is fairly simple for servers as well. The creation of the display
and binding to a socket are separate, to give you time to configure the display
before any clients are able to connect to it. Here's another minimal example
program:

对于 server 来说，整个过程同样很简单。display 的创建与 socket 的绑定是分开的。为的是确保有
足够的时间在 client 能够连接到这个 display 之前配置它。 如下示例程序： 

```c
#include <stdio.h>
#include <wayland-server.h>

int
main(int argc, char *argv[])
{
    struct wl_display *display = wl_display_create();
    if (!display) {
        fprintf(stderr, "Unable to create Wayland display.\n");
        return 1;
    }

    const char *socket = wl_display_add_socket_auto(display);
    if (!socket) {
        fprintf(stderr, "Unable to add socket to Wayland display.\n");
        return 1;
    }

    fprintf(stderr, "Running Wayland display on %s\n", socket);
    wl_display_run(display);

    wl_display_destroy(display);
    return 0;
}
```

Let's compile and run this, too:

让我们再次尝试编译并运行它：

```sh
$ cc -o server server.c -lwayland-server
$ ./server &
Running Wayland display on wayland-1
$ WAYLAND_DISPLAY=wayland-1 ./client
Connection established!
```

Using `wl_display_add_socket_auto` will allow libwayland to decide the name for
the display automatically, which defaults to `wayland-0`, or `wayland-$n`,
depending on if any other Wayland compositors have sockets in
`$XDG_RUNTIME_DIR`. However, as with the client, you have some other options for
configuring the display:

使用 `wl_display_add_socket_auto` 将允许 libwayland 自行决定 display 的名称，默认为
`wayland-0` 或 `wayland-$n`，具体名字取决于其他 Wayland compositor 是否在 `$XDG_RUNTIME_DIR` 
中已经有其他的 socket 存在。当然，与 client 一样，你还有其他一些用于配置 display 的选项：

```c
int wl_display_add_socket(struct wl_display *display, const char *name);

int wl_display_add_socket_fd(struct wl_display *display, int sock_fd);
```

After adding the socket, calling `wl_display_run` will run libwayland's internal
event loop and block until `wl_display_terminate` is called. What's this event
loop? Turn the page and find out!

添加 socket 后，调用 `wl_display_run` 将运行 libwayland 内部的 event loop 并阻塞，直到调
用 `wl_display_terminate`。 event loop 是怎样工作的？ 翻开下一页，一探究竟！ 

原文链接:https://wayland-book.com/

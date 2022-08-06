---
layout: post
title:  "6.2 Wayland book Shared memory buffers"
date:   2022-08-05 13:43:29 +0800
categories: waylandbook
---
# Shared memory buffers

The simplest means of getting pixels from client to compositor, and the only one
enshrined in `wayland.xml`, is `wl_shm` &mdash; shared memory. Simply put, it 
allows you to transfer a file descriptor for the compositor to mmap with 
`MAP_SHARED`, then share pixel buffers out of this pool. Add some simple 
synchronization primitives to keep everyone from fighting over each buffer, and 
you have a workable &mdash; and portable &mdash; solution.

compositor 从客户端获取对应像素内容最简单的，唯一保存在 `wayland.xml` 中的方法是 `wl_shm`；共享内存机制。
简单来说，它允许你传输一个使用 `MAP_SHARED` flag 的 mmap 创建的 fd 到 compositor，然后通过这个 pool 共享像素 buffer。
添加一些简单的同步原语保证每个缓冲区的竞争，（如上）就构成了一个可行可移植的解决方案。 

## Binding to wl_shm

The registry global listener explained in chapter 5.1 will advertise the
`wl_shm` global when it's available. Binding to it is fairly straightforward.
Extending the example given in chapter 5.1, we get the following:

在第 5.1 章中讲述的 listener global 对象将在 `wl_shm` global 对象可用的第一时间发布（消息）。
绑定上（wl_shm）相当简单。扩展 5.1 章的示例代码，如下：

```
struct our_state {
    // ...
    struct wl_shm *shm;
    // ...
};

static void
registry_handle_global(void *data, struct wl_registry *registry,
		uint32_t name, const char *interface, uint32_t version)
{
    struct our_state *state = data;
    if (strcmp(interface, wl_shm_interface.name) == 0) {
        state->shm = wl_registry_bind(
            wl_registry, name, &wl_shm_interface, 1);
    }
}

int
main(int argc, char *argv[])
{
    struct our_state state = { 0 };
    // ...
    wl_registry_add_listener(registry, &registry_listener, &state);
    // ...
}
```

Once bound, we can optionally add a listener via `wl_shm_add_listener`. The
compositor will advertise its supported pixel formats via this listener. The
full list of possible pixel formats is given in `wayland.xml`. Two formats are
required to be supported: `ARGB8888`, and `XRGB8888`, which are 24-bit color,
with and without an alpha channel respectively.

绑定后，我们可以通过 `wl_shm_add_listener` 添加一个 listener。Compositor 将通过这个监听器传递它
所支持的像素格式。（像素格式的）完整列表在 `wayland.xml` 中给出。两种格式被要求必须支持：`ARGB8888` 和
`XRGB8888`，24位颜色，支持或不支持 alpha 通道。

## Allocating a shared memory pool


A combination of POSIX `shm_open` and random file names can be utilized to
create a file suitable for this purpose, and `ftruncate` can be utilized to
bring it up to the appropriate size. The following boilerplate may be freely
used under public domain or CC0:

使用 POSIX 函数 `shm_open` 以及随机文件名可以创建一个（用于支持shared-memory）的文件，
然后使用 `ftruncate` 将它调整到合适大小。非商业目的，遵循 CC0 的情况下可随意使用如下示例代码：

```
#define _POSIX_C_SOURCE 200112L
#include <errno.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <time.h>
#include <unistd.h>

static void
randname(char *buf)
{
	struct timespec ts;
	clock_gettime(CLOCK_REALTIME, &ts);
	long r = ts.tv_nsec;
	for (int i = 0; i < 6; ++i) {
		buf[i] = 'A'+(r&15)+(r&16)*2;
		r >>= 5;
	}
}

static int
create_shm_file(void)
{
	int retries = 100;
	do {
		char name[] = "/wl_shm-XXXXXX";
		randname(name + sizeof(name) - 7);
		--retries;
		int fd = shm_open(name, O_RDWR | O_CREAT | O_EXCL, 0600);
		if (fd >= 0) {
			shm_unlink(name);
			return fd;
		}
	} while (retries > 0 && errno == EEXIST);
	return -1;
}

int
allocate_shm_file(size_t size)
{
	int fd = create_shm_file();
	if (fd < 0)
		return -1;
	int ret;
	do {
		ret = ftruncate(fd, size);
	} while (ret < 0 && errno == EINTR);
	if (ret < 0) {
		close(fd);
		return -1;
	}
	return fd;
}
```

Hopefully the code is fairly self-explanatory (famous last words). Armed with
this, the client can create a shared memory pool fairly easily. Let's say, for
example, that we want to show a 1920x1080 window. We'll need two buffers for
double-buffering, so that'll be 4,147,200 pixels. Assuming the pixel format is
`WL_SHM_FORMAT_XRGB8888`, that'll be 4 bytes to the pixel, for a total pool size
of 16,588,800 bytes. Bind to the `wl_shm` global from the registry as explained
in chapter 5.1, then use it like so to create an shm pool which can hold these
buffers:

这段代码想必是不需要解释的，如上操作，客户端可以简单的创建一个共享内存池。比方说，我们想
创建一个 1920x1080 大小的窗口。我们需要两个缓冲区，实现双缓冲机制，pixel大小将是 4147200。
假设像素格式是 `WL_SHM_FORMAT_XRGB8888`，对于总的内存池大小，一个像素是 4 字节，即 16588800 字节。
如 5.1 章所述，从 registry 中绑定 `wl_shm` global 对象，然后使用它创建一个需要大小的 shm pool。

```
const int width = 1920, height = 1080;
const int stride = width * 4;
const int shm_pool_size = height * stride * 2;

int fd = allocate_shm_file(shm_pool_size);
uint8_t *pool_data = mmap(NULL, shm_pool_size,
    PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

struct wl_shm *shm = ...; // Bound from registry
struct wl_shm_pool *pool = wl_shm_create_pool(shm, fd, shm_pool_size);
```

## Creating buffers from a pool

Once word of this gets to the compositor, it will `mmap` this file descriptor as
well. Wayland is asynchronous, though, so we can start allocating buffers from
this pool right away. Since we allocated space for two buffers, we can assign
each an index and convert that index into a byte offset in the pool. Equipped
with this information, we can create a `wl_buffer`:

一旦这些 request 到达合成器，合成器会对这个文件描述符进行 `mmap` 操作。并且由于 Wayland 是异步的，我们可以即刻从该 pool 里面
分配缓冲区。因为我们分配了两个缓冲区大小（显示双缓冲机制，非常常见）的空间，我们需要为每个缓冲区分配一个索引并将该索引转换为 pool 中的字节偏移量。有了这些信息，
我们就可以创建一个 `wl_buffer`了：

```
int index = 0;
int offset = height * stride * index;
struct wl_buffer *buffer = wl_shm_pool_create_buffer(pool, offset,
    width, height, stride, WL_SHM_FORMAT_XRGB8888);
```

We can write an image to this buffer now as well. For example, to set it to
solid white:

我们现在也可以将图像数据写入此缓冲区。例如，要将其设置为纯白色：

```
uint32_t *pixels = (uint32_t *)&pool_data[offset];
memset(pixels, 0, width * height * 4);
```

Or, for something more interesting, here's a checkerboard pattern:

或者，更有趣一点，我们填充 buffer 为一个棋盘：

```
uint32_t *pixels = (uint32_t *)&pool_data[offset];
for (int y = 0; y < height; ++y) {
  for (int x = 0; x < width; ++x) {
    if ((x + y / 8 * 8) % 16 < 8) {
      pixels[y * width + x] = 0xFF666666;
    } else {
      pixels[y * width + x] = 0xFFEEEEEE;
    }
  }
}
```

With the stage set, we'll attach our buffer to our surface, mark the whole
surface as damaged[^1], and commit it:

设置完如上步骤，我们将缓冲区附加到我们的表面，将整个表面标记为损坏 [^1]，并提交它：

```c
wl_surface_attach(surface, buffer, 0, 0);
wl_surface_damage(surface, 0, 0, UINT32_MAX, UINT32_MAX);
wl_surface_commit(surface);
```

If you were to apply all of this newfound knowledge to writing a Wayland client
yourself, you may arrive at this point confused when your buffer is not shown
on-screen. We're missing a critical final step &mdash; assigning your surface a
role.

如果你将这些新的知识应用于自己编写的 Wayland 客户端，你可能会困惑仍然没有把缓冲区内容显示在屏幕上。
我们错过了关键的最后一步 &mdash; 为你的 surface 设置角色。

[^1]: "Damaged" meaning "this area needs to be redrawn"

[^1]: "Damaged" 意思是 "这块区域需要重绘"


## wl_shm on the server

Before we get there, however, the server-side part of this deserves note.
libwayland provides some helpers to make using `wl_shm` easier. To configure it
for your display, it only requires the following:

在我们探究 surface 角色之前，我们需要讲解一下服务端的 wl_shm 接口。libwayland 提供了一些帮助函数来更加简单的使用 `wl_shm`。
要为你的 display 对象配置这些，只需要如下操作：

```
int
wl_display_init_shm(struct wl_display *display);

uint32_t *
wl_display_add_shm_format(struct wl_display *display, uint32_t format);
```

The former creates the global and rigs up the internal implementation, and the
latter adds a supported pixel format (remember to at least add ARGB8888 and
XRGB8888). Once a client attaches a buffer to one of its surfaces, you can pass
the buffer resource into `wl_shm_buffer_get` to obtain a `wl_shm_buffer`
reference, and utilize it like so:

前者完成创建全局对象和设置内部实现，后者添加支持的像素格式（至少添加 ARGB8888 和 XRGB8888）。
一旦客户端将缓冲区附加到其表面，你可以使用 buffer resource 参数通过 `wl_shm_buffer_get` 得到 `wl_shm_buffer` 引用，并像这样使用它：

```
void
wl_shm_buffer_begin_access(struct wl_shm_buffer *buffer);

void
wl_shm_buffer_end_access(struct wl_shm_buffer *buffer);

void *
wl_shm_buffer_get_data(struct wl_shm_buffer *buffer);

int32_t
wl_shm_buffer_get_stride(struct wl_shm_buffer *buffer);

uint32_t
wl_shm_buffer_get_format(struct wl_shm_buffer *buffer);

int32_t
wl_shm_buffer_get_width(struct wl_shm_buffer *buffer);

int32_t
wl_shm_buffer_get_height(struct wl_shm_buffer *buffer);
```

If you guard your accesses to the buffer data with `begin_access` and
`end_access`, libwayland will take care of locking for you

如果你在访问 buffer 数据的时候调用 `begin_access` 和 `end_access`, libwayland 将会对 buffer 进行锁保护。



原文链接:https://wayland-book.com/

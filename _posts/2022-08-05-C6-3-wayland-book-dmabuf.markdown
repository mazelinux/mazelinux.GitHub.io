---
layout: post
title:  "6.3 Wayland book Linux dmabuf"
date:   2022-08-05 13:44:29 +0800
categories: jekyll update
---
# Linux dmabuf

<!-- TODO: Move me to an appendix -->

Most Wayland compositors do their rendering on the GPU, and many Wayland clients
do their rendering on the GPU as well. With the shared memory approach, sending
buffers from the client to the compositor in such cases is very inefficient, as
the client has to read their data from the GPU to the CPU, then the compositor
has to read it from the CPU back to the GPU to be rendered.

大多数 Wayland compostior 使用 GPU 完成渲染（也就是常说的合成），许多 Wayland client 也使用 GPU 处理渲染逻辑。
在上述 case 中，使用共享内存的方式，从客户端发送缓冲区到合成器非常低效，因为客户端需要先把他们的渲染数据从 GPU 读到 CPU，
发给合成器，然后合成器再把这些数据从 CPU 写回 GPU，才能完成合成器内部的合成逻辑。

The Linux DRM (Direct Rendering Manager) interface (which is also implemented on
some BSDs) provides a means for us to export handles to GPU resources. Mesa, the
predominant implementation of userspace Linux graphics drivers, implements a
protocol that allows EGL users to transfer handles to their GPU buffers from the
client to the compositor for rendering, without ever copying data to the GPU.

Linux DRM 接口（直接渲染管理器，也在一些BSD版本实现）提供了一种将 handle 作为 GPU 资源导出的手段
（通过 handle 完成 GPU 和 DPU 共享buffer，而不存在 GPU 到 CPU 的频繁拷贝）。Mesa 是 Linux 图形驱动程序的用户空间的主要实现，
它实现了一套协议，支持 EGL 用户将 GPU缓冲区对应的 handle 从客户端传给合成器，以完合成器的合成动作（以上逻辑不存在数据拷贝）。

The internals of how this protocol works are out of scope for this book and
would be more appropriate for resources which focus on Mesa or Linux DRM in
particular. However, we can provide a short summary of its use.

关于此协议的内部工作方式超出了本书的范畴，它更适合专注与 Mesa 或者 Linux DRM 的（学者）。
当然，我们可以提供一个如何使用的简单小节。



1. Use `eglGetPlatformDisplayEXT` in concert with `EGL_PLATFORM_WAYLAND_KHR` to
   create an EGL display.
2. Configure the display normally, choosing a config appropriate to your
   circumstances with `EGL_SURFACE_TYPE` set to `EGL_WINDOW_BIT`.
3. Use `wl_egl_window_create` to create a `wl_egl_window` for a given
   `wl_surface`.
4. Use `eglCreatePlatformWindowSurfaceEXT` to create an `EGLSurface` for a
   `wl_egl_window`.
5. Proceed using EGL normally, e.g. `eglMakeCurrent` to make current the EGL
   context for your surface and `eglSwapBuffers` to send an up-to-date buffer to
   the compositor and commit the surface.
   
1. 调用 `eglGetPlatformDisplayEXT`，platform 指定为 `EGL_PLATFORM_WAYLAND_KHR`，
创建一个 EGL display 对象。
2. 正确配置 display 对象，在基于 `EGL_SURFACE_TYPE` 为 `EGL_WINDOW_BIT` 的情况下，
选择一个适合你的配置。
3. 调用 `wl_egl_window_create` 函数创建对应 `wl_surface` 的 `wl_egl_window`。
4. 调用 `eglCreatePlatformWindowSurfaceEXT` 函数创建对应 `wl_egl_window` 的 `EGLSurface`。
5. 继续正常调用 EGL 接口，例如，（调用） `eglMakeCurrent` 绑定当前 EGL context 到你的 surface，
（调用） `eglSwapBuffers` 发送最新的缓冲区到 compositor，并且执行 commit 命令。

Should you need to change the size of the `wl_egl_window` later, use
`wl_egl_window_resize`.

如果需要改变 `wl_egl_window` 的大小，使用 `wl_egl_window_resize`。

## But I really want to know about the internals

Some Wayland programmers who don't use libwayland complain that this approach
ties Mesa and libwayland tightly together, which is true. However, untangling
them is not impossible &mdash; it just requires a lot of work for you in the 
form of implementing `linux-dmabuf` yourself. Consult the Wayland extension XML
for details on the protocol, and Mesa's implementation at
`src/egl/drivers/dri2/platform_wayland.c` (at the time of writing). Good luck
and godspeed.

一些不使用 libwayland 的 Wayland 程序员抱怨说这种方式使得 Mesa 和 libwayland 耦合严重，
确实如此。然而，解耦也不是不可能 &mdash; 你需要做大量的工作，以自己实现 `linux-dmabuf`。
可以参阅 Wayalnd 扩展 XML 协议以获得更多细节内容，Mesa的实现可以在 `src/egl/drivers/dri2/platform_wayland.c` 找到。
祝你好运~


## For the server

Unfortunately, the details for the compositor are both complicated and
out-of-scope for this book. I can point you in the right direction, however:
the wlroots implementation (found at `types/wlr_linux_dmabuf_v1.c` at the time
of writing) is straightforward and should set you on the right path.

不幸的是，Compositor 相关（dmabuf）的细节实现太过复杂，超出了本书范畴。但是我可以为你指明方向：
wlroots 的实现（可以在 types/wlr_linux_dmabuf_v1.c 找到）非常简单，是一个正确的参考实现。

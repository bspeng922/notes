# Ubuntu 14.04 sublime设置
### 安装：
**下载sublime text3**
[http://www.sublimetext.com/3](http://www.sublimetext.com/3)

**安装插件：**
*安装Package Control*
[https://packagecontrol.io/installation](https://packagecontrol.io/installation)
*其他插件*
[https://packagecontrol.io/](https://packagecontrol.io/)
打开sublime，按ctrl+shift+p，输入pi，选择Install Package后回车，输入插件名称就可以安装相应插件:

+ svn     #SVN插件
+ sublimerge  #svn diff工具
+ Sidebarenhancement  #边栏
+ converttoutf8   #中文乱码
+ codecs33    #中文乱码
+ SublimeGit  #git插件
+ gitgutter   #git插件
+ Anaconda    #代码引用跳转

### 输入中文：
**安装相关依赖**
```shell
$ sudo apt-get install build-essential libgtk2.0-dev
```

**进入用户主目录**
```shell
$ cd ～
```

**新建文件**
```shell
$ touch sublime_imfix.c
```

**编辑并保存文件**
```shell
$ vim sublime_imfix.c
```

```c
/*
 * sublime-imfix.c
 * Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
 * By Cjacker Huang <jianzhong.huang at i-soft.com.cn> *
 *
 * gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
 * LD_PRELOAD=./libsublime-imfix.so sublime_text
 */
\#include <gtk/gtk.h>
\#include <gdk/gdkx.h>

typedef GdkSegment GdkRegionBox;

struct _GdkRegion
{
    long size;
    long numRects;
    GdkRegionBox *rects;
    GdkRegionBox extents;
};

GtkIMContext *local_context;

void
gdk_region_get_clipbox (const GdkRegion *region,
                        GdkRectangle    *rectangle)
{
    g_return_if_fail (region != NULL);
    g_return_if_fail (rectangle != NULL);

    rectangle->x = region->extents.x1;
    rectangle->y = region->extents.y1;
    rectangle->width = region->extents.x2 - region->extents.x1;
    rectangle->height = region->extents.y2 - region->extents.y1;
    GdkRectangle rect;
    rect.x = rectangle->x;
    rect.y = rectangle->y;
    rect.width = 0;
    rect.height = rectangle->height;

    //The caret width is 2;
    //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
    if (rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
        gtk_im_context_set_cursor_location(local_context, rectangle);
    }
}

//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.

static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
    XEvent *xev = (XEvent *)xevent;

    if (xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
        GdkWindow *win = g_object_get_data(G_OBJECT(im_context), "window");

        if (GDK_IS_WINDOW(win)) {
            gtk_im_context_set_client_window(im_context, win);
        }
    }

    return GDK_FILTER_CONTINUE;
}

void gtk_im_context_set_client_window (GtkIMContext *context,
                                       GdkWindow    *window)
{
    GtkIMContextClass *klass;
    g_return_if_fail (GTK_IS_IM_CONTEXT (context));
    klass = GTK_IM_CONTEXT_GET_CLASS (context);

    if (klass->set_client_window) {
        klass->set_client_window (context, window);
    }

    if (!GDK_IS_WINDOW (window)) {
        return;
    }

    g_object_set_data(G_OBJECT(context), "window", window);
    int width = gdk_window_get_width(window);
    int height = gdk_window_get_height(window);

    if (width != 0 && height != 0) {
        gtk_im_context_focus_in(context);
        local_context = context;
    }

    gdk_window_add_filter (window, event_filter, context);
}
```

**编译该文件**
```shell
$ gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
```

**拷贝库文件到sublime主目录**
```shell
$ sudo cp libsublime-imfix.so /opt/sublime_text/
```

**在终端测试**
```shell
$ LD_PRELOAD=./libsublime-imfix.so subl
```

**修改文件/usr/bin/subl的内容**
```shell
$ sudo gedit /usr/bin/subl
```
将
```shell
#!/bin/sh
exec /opt/sublime_text/sublime_text "$@"
```
修改为
```shell
#!/bin/sh
LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"
```
此时，在命令中执行 subl 将可以使用搜狗for linux的中文输入


**编辑/usr/share/applications/sublime_text.desktop文件**
将[Desktop Entry]中的字符串
Exec=/opt/sublime_text/sublime_text %F
修改为
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"

将[Desktop Action Window]中的字符串
Exec=/opt/sublime_text/sublime_text -n
修改为
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"

将[Desktop Action Document]中的字符串
Exec=/opt/sublime_text/sublime_text --command new_file
修改为
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"

> **注意：**
> 修改时请注意双引号"",否则会导致不能打开带有空格文件名的文件。

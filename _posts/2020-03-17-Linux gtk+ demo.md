---
layout: post
title: "Linux GTK+-3 Demo"
tagline: "Linux GTK+-3 Demo"
categories: gtk
author: "kekxv"
---

# GTK 简介

GTK是一款开源的、面向多平台的GUI工具箱，其英文全称为GIMP Toolkit。最初是Peter Mattis 和 Spencer Kimball 为GNU Image Manipulation Program (GIMP)编写。在后续的发展中，它已经成为通用的GUI库，应用于越来越多的程序，Linux平台的图形应用程序的半壁江山都是使用GTK编写的。

# 使用

目前GTK已经基本稳定，然而由于越来越多的其他GUI库出现，以及HTML5的统治，目前GTK系列的需求已经较少。

在比较早之前，就一直在关注GTK，然后当年水平有限，以及精力以及方向并不在这方面，所以也就并没有怎么了解以及使用，最近有个项目需要用到界面，然后为了方便调试（主要是开发电脑与实际电脑不在同一设备上，也不是同一架构，不想要来回跑动，所以使用了SSH X11 转发，而GTK可以很好地支持）。

废话不多说，上代码看看：

```cpp
//
// Created by caesar kekxv on 2020/3/18.
//
#include <iostream>
#include <vector>
#include <opencv2/opencv.hpp>
#include <gtk/gtk.h>
#include <thread>
#include <atomic>

#if defined(LINUX) || defined(linux)
#include <X11/Xlib.h>
#endif
#ifdef WIN32
#include "getopt.h"
#else
#include <getopt.h>
#endif

using namespace std;
using namespace cv;

int X_WIDTH       =   640;
int X_HEIGHT      =   480;

VideoCapture cap;
cv::Mat last_frame;
int fps = 30;

// 是否关闭
std::atomic<bool> stoped{false};
GMutex _mutex;
std::thread video_task;

GtkWidget *image = nullptr;
/* Surface to store current scribbles */
static cairo_surface_t *surface = nullptr;
/**
 * OpenCV Mat 格式转换为 GTK GdkPixbuf 格式
 */
GdkPixbuf *MatToGdkPixbuf(Mat inMat) {
    // IplImage -> GdkPixbuf
    GdkPixbuf *src = gdk_pixbuf_new_from_data(
            (const guchar *) inMat.data, GDK_COLORSPACE_RGB, false, 8, inMat.size().width, inMat.size().height, inMat.step, nullptr, nullptr);
    return src;
}

/**
 * 清理画布
 */
static void clear_surface() {
    if (surface == nullptr)return;
    cairo_t *cr;
    cr = cairo_create(surface);
    cairo_set_source_rgb(cr, 1, 1, 1);
    cairo_paint(cr);
    cairo_destroy(cr);
}

/* Create a new surface of the appropriate size to store our scribbles */
static gboolean configure_event_cb(GtkWidget *widget,
                   GdkEventConfigure *event,
                   gpointer data) {
    if (surface)
        cairo_surface_destroy(surface);
    surface = gdk_window_create_similar_surface(gtk_widget_get_window(widget),
                                                CAIRO_CONTENT_COLOR,
                                                gtk_widget_get_allocated_width(widget),
                                                gtk_widget_get_allocated_height(widget));
    /* Initialize the surface to white */
    clear_surface();
    /* We've handled the configure event, no need for further processing. */
    return TRUE;
}

/**
 * 关闭窗口释放
 */
static void close_window() {
    // std::unique_lock<std::mutex> _lock{m_lock};
    if (surface)
        cairo_surface_destroy(surface);
    surface = nullptr;
}


/* Redraw the screen from the surface. Note that the ::draw
 * signal receives a ready-to-be-used cairo_t that is already
 * clipped to only draw the exposed areas of the widget
 */
static gboolean draw_event(GtkWidget *widget, cairo_t *cr) {
    GdkWindow *win;
    win = gtk_widget_get_window(widget);
    g_mutex_lock(&_mutex);
    if (!last_frame.empty()) {
        auto src = MatToGdkPixbuf(last_frame);
        g_mutex_unlock(&_mutex);
        if (src == nullptr) {
            return FALSE;
        }
        gdk_cairo_set_source_pixbuf(cr, src, 0, 0);
        cairo_paint(cr);
        cairo_fill(cr);
        g_object_unref(src);
    } else
        g_mutex_unlock(&_mutex);
    return TRUE;
}

/**
 * OpenCV 摄像头 线程
 */
void video_task_run(GtkWidget *_image) {
    while (!stoped.load() && cap.isOpened()) {
        Mat frame;
        cap >> frame;
        if (frame.empty())break;

        resize(frame, frame, Size(X_WIDTH, X_HEIGHT));
        cvtColor(frame, frame, COLOR_BGR2RGB);
        if (surface == nullptr) {
            usleep(1000 / fps * 1000);
            continue;
        }
        // GTK 锁
        g_mutex_lock(&_mutex);
        if (surface == nullptr) {
            g_mutex_unlock(&_mutex);
            break;
        }
        last_frame = frame;
        // 信号通知
        g_idle_add((GSourceFunc) gtk_widget_queue_draw, (void *) _image);
        // gdk_threads_add_idle((GSourceFunc) gtk_widget_queue_draw, (void *) _image);
        g_mutex_unlock(&_mutex);

        usleep(1000 / fps * 800);
        g_idle_remove_by_data((void *) _image);
    }
    // logger::instance()->d(__FILENAME__, __LINE__, "视频读取完毕");
}

static void activate(GtkApplication *app, gpointer user_data) {
    GtkWidget *window;
    window = gtk_application_window_new(app);
    gtk_window_set_title(GTK_WINDOW (window), "Window");
    gtk_window_set_default_size(GTK_WINDOW (window), X_WIDTH, X_HEIGHT);
    gtk_window_set_position( GTK_WINDOW(window), GTK_WIN_POS_CENTER_ALWAYS );

    g_signal_connect (window, "destroy", G_CALLBACK(close_window), NULL);

    image = gtk_drawing_area_new();

    gtk_widget_set_size_request(image, X_WIDTH, X_HEIGHT);
    gtk_container_add(GTK_CONTAINER (window), image);

    /* Signals used to handle the backing surface */
    g_signal_connect (image, "draw",
                      G_CALLBACK(draw_event), NULL);

    g_signal_connect (image, "configure-event",
                      G_CALLBACK(configure_event_cb), NULL);

    video_task = thread(video_task_run, image);
    video_task.detach();

    gtk_widget_show_all(window);
}


int main(int argc, char *argv[]) {
#if defined(LINUX) || defined(linux)
    setenv("DISPLAY", "localhost:10.0", 1);
    // setenv("DISPLAY", ":1", 1);
    logger::instance()->i(__FILENAME__, __LINE__, "XOpenDisplay : %s", XOpenDisplay(nullptr) ? "True" : "False");
#endif

    cap.open(0);
    if (cap.isOpened()) {
        fps = cap.get(CAP_PROP_FPS);
        X_WIDTH = cap.get(CAP_PROP_FRAME_WIDTH);
        X_HEIGHT = cap.get(CAP_PROP_FRAME_HEIGHT);
        cap.read(last_frame);
    }
    GtkApplication *app;
    int status;

    app = gtk_application_new("com.example", G_APPLICATION_FLAGS_NONE);
    g_signal_connect (app, "activate", G_CALLBACK(activate), NULL);
    status = g_application_run(G_APPLICATION (app), 1, argv);
    g_object_unref(app);

    return status;
}

```

如果有什么错误或者不明白的地方，可以留言告诉我。

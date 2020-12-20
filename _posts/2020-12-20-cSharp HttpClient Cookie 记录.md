---
layout: post 
title: "cSharp HttpClient Cookie 记录"
tagline: "cSharp HttpClient Cookie 记录"
categories: cSharp
---

使用`CookieContainer`自动管理你的`HttpClient` `Cookie`。

因为业务需要，客户端需要进行登录，登录的会话`token`保留在`cookie`呢，所以需要让`HttpClient`携带上`Cookie`，本文作为一个记录。

```java
class HttpClientTool {
    /**
     * 如果全局cookie，可以使用静态变量，如果不是全局，可只作为内部变量
     */
    private static CookieContainer cookies = new CookieContainer();

    public static void main(String[] args) {
        HttpClientHandler handler = new HttpClientHandler();
        // 开启cookie
        handler.UseCookies = true;
        // 使用 CookieContainer 管理
        handler.CookieContainer = cookies;
        // 将 HttpClientHandler 传入HttpClient，之后发送接收请求都会通过`cookies`进行cookie管理
        HttpClient httpClient = new HttpClient(handler);
    }
}
```

`MacOs` 版本的 `Rider` 居然不支持 `WinForm` ，还得开个虚拟机测试。啧

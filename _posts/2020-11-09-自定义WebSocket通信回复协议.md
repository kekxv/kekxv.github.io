---
layout: post
title: "自定义WebSocket通信回复协议"
tagline: "自定义WebSocket通信回复协议"
categories: WebSocket
author: "kekxv"
---

在有些情况下，我们在使用 `WebSocket` 通信的时候，需要对方进行回应，但由于 `WebSocket` 是`全双工`交互，所以不能像 `http` 协议一样，发送之后能够得到对应的回复，为了能够很好的获取到对应的回复，设计了一个简单的协议，各位有需要的话，可以参考看看。

协议的原理很简单，即：`记录消息ID`。

举个例子🌰：

> JavaScript 发送数据需要对方进行回复；那么对发送的数据进行封装：
>
> ```JavaScript
> sendJson = function (json) {
>     return new Promise(((resolve, reject) => {
>         let dataPackage = {
>             _message_id: json.$_message_id || {web_id: 0, sdk_id: 0},
>             data: json,
>         }
>         dataPackage._message_id.web_id = _message_id++;
>         if (_message_id > _callback_max_id) {
>             _message_id = 0;
>         }
>         try {
>             webSocket.send(JSON.stringify(dataPackage));
>             _callback[dataPackage._message_id.web_id] = {
>                 resolve: resolve, reject: reject
>             }
>         } catch (e) {
>             reject(e);
>         }
>     }));
> }
> webSocket.onmessage = async function (event) {
>   try {
>       let dataPackage = JSON.parse(event.data);
>       let _message_id = dataPackage._message_id;
>       let json = dataPackage.data;
>       json.$_message_id = _message_id;
>       let message;
>       try {
>           if (_message_id && _message_id.web_id > 0) {
>               let func = _callback[_message_id.web_id];
>               delete _callback[_message_id.web_id];
>               _message_id.web_id = 0;
>               json.$_message_id.web_id = 0;
>               if (json.code >= 0) {
>                   func.resolve(json);
>               } else {
>                   func.reject(json);
>               }
>           } else {
>               message = option && option.onmessage && await option.onmessage(json);
>           }
>       } catch (e) {
>           message = {
>               code: -1,
>               message: typeof e == "string" ? e : JSON.stringify(e),
>               data: null,
>           };
>       } finally {
>           let dataPackage = {
>               _message_id: _message_id,
>               data: message
>           }
>           delete message.$_message_id;
>           if (dataPackage._message_id && dataPackage._message_id.sdk_id) {
>               self.sendJsonBack(dataPackage);
>           }
>       }
>   } catch (e) {
> 
>   }
> };
> 
> ```
> 
> 变量说明：
> 1. `web_id` 和 `sdk_id` 在这里分别表示网页端消息 id 和 `sdk`(我这边另一头是作为`sdk`提供，所以命名为 `sdk_id`) 端消息ID。
> 1. `webSocket` 为 `WebSocket` 对象，用于接收和发送 `WebSocket` 数据。
> 1. `_callback` 与 `_callback_max_id` 分别为`记录回调函数`，以及`最大回调记录`，可以在接收到回复的情况下进行回调，从而与发送关联。
> 1. `onmessage` 为 `WebSocket`接收数据回调，通过接收的 `_message_id` 内 `web_id` 和 `sdk_id` （`JavaScript`端主要是 `web_id`），然后根据 `webSocket` 进行对应数据回调。
> 1. 为保证对方也能够得到消息的回复，所以在 `finally` 也进行 `dataPackage._message_id.sdk_id` 即 `sdk_id` 的判定，如果对方发送的数据包含 `dataPackage._message_id.sdk_id` ，那么表示这个包为发送包，非回应包，需要 `JavaScript` 这端进行回应。


`JavaScript` 端完整例子：
```javascript
const _callback_max_id = 1000000000;
let WsApi = function (option) {
    let self = this;
    this._initWebSocket(option || {});
}

WsApi.prototype = {
    _initWebSocket(option) {
        let self = this;
        let _message_id = 0;
        let _callback = [];
        let webSocket = new WebSocket(
            option.url || `${window.location.protocol.replace("http", "ws")}//${window.location.host}/ws`
        );

        this.sendJson = function (json) {
            return new Promise(((resolve, reject) => {
                let dataPackage = {
                    _message_id: json.$_message_id || {web_id: 0, sdk_id: 0},
                    data: json,
                }
                delete json.$_message_id;
                dataPackage._message_id.web_id = _message_id++;
                if (_message_id > _callback_max_id) {
                    _message_id = 0;
                }
                try {
                    webSocket.send(JSON.stringify(dataPackage));
                    _callback[dataPackage._message_id.web_id] = {
                        resolve: resolve, reject: reject
                    }
                } catch (e) {
                    reject(e);
                }
            }));
        }
        this.sendJsonBack = function (dataPackage) {
            try {
                webSocket.send(JSON.stringify(dataPackage));
            } catch (e) {
                console.log(e);
            }
        }
        webSocket.onerror = function (event) {
            console.log(event)
        };
        webSocket.onopen = function (event) {
            console.log(event);
        };
        webSocket.onclose = function (event) {
            console.log(event)
            setTimeout(() => {
                // TODO 循环调用
                // self._initWebSocket(option);
            }, 1000);
        };
        webSocket.onmessage = async function (event) {
            try {
                let dataPackage = JSON.parse(event.data);
                let _message_id = dataPackage._message_id;
                let json = dataPackage.data;
                json.$_message_id = _message_id;
                let message;
                try {
                    if (_message_id && _message_id.web_id > 0) {
                        let func = _callback[_message_id.web_id];
                        delete _callback[_message_id.web_id];
                        _message_id.web_id = 0;
                        json.$_message_id.web_id = 0;
                        if (json.code >= 0) {
                            func.resolve(json);
                        } else {
                            func.reject(json);
                        }
                    } else {
                        message = option && option.onmessage && await option.onmessage(json);
                    }
                } catch (e) {
                    message = {
                        code: -1,
                        message: typeof e == "string" ? e : JSON.stringify(e),
                        data: null,
                    };
                } finally {
                    let dataPackage = {
                        _message_id: _message_id,
                        data: message
                    }
                    delete message.$_message_id;
                    if (dataPackage._message_id && dataPackage._message_id.sdk_id) {
                        self.sendJsonBack(dataPackage);
                    }
                }
            } catch (e) {

            }
        };
    },
}

export default WsApi
```

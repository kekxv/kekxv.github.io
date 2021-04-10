---
layout: post
title: "Web 版图像拼接(OpenCV)"
tagline: "Web 版图像拼接(OpenCV)"
categories: opencv
author: "kekxv"
---

在某些情况下，我们需要将两张图像合并为一张，这时候我们会选择类似 `PhotoShow` 等画图软件进行拼接。不过我想偷个懒，自动拼接。
所以查找了一些资料之后，开发了 `Web版本的图像拼接` 接口。

## 开始之前

用到的 web 端使用的是[kProxyCpp](/assets/imgs/old/?id=3 "kProxyCpp") ，图像处理用的是 `OpenCV`，技术实现参考了：[OpenCV探索之路（二十四）图像拼接和图像融合技术](https://www.cnblogs.com/skyfsm/p/7411961.html "OpenCV探索之路（二十四）图像拼接和图像融合技术")（这篇文章有更详细的介绍，有兴趣的可以看看）。
源码已经上传至 `Github` ，仓库为：[kProxyCpp](https://github.com/kekxv/kProxyCpp "kProxyCpp")

## 代码实现

### http端实现

使用的是[kProxyCpp](https://github.com/kekxv/kProxyCpp "kProxyCpp")，比较简单点：

```c++
// 初始化
kHttpd::Init();
// 开启一个服务
kHttpd kProxy("网站目录", "线程池数量");
// 绑定路径 POST /MergePhoto
kProxy.set_cb("POST", "/MergePhoto",[](void *kClient, const std::vector<unsigned char> &data, const std::string &url_path,const std::string &method, int type, void *arg) -> int {
	// 如果开启了WebSocket的情况下，并且是WebSocket链接，则 Type 将会 >=0;
	if (type == -1) {
		// 在这里进行业务操作
		…………
	} else {
		return -1;
	}
});
// 开始监听
kProxy.listen(20, port, ip.c_str());
```

这样我们就绑定了路径 `POST /MergePhoto` ，然后结合我们的业务进行操作。

### 图片拼接

原理就是
1. 对每幅图进行特征点提取
2. 对对特征点进行匹配
3. 进行图像配准
4. 把图像拷贝到另一幅图像的特定位置
5. 对重叠边界进行特殊处理

参考的原文（[OpenCV探索之路（二十四）图像拼接和图像融合技术](https://www.cnblogs.com/skyfsm/p/7411961.html "OpenCV探索之路（二十四）图像拼接和图像融合技术")）有更加详细的介绍，我这边就不重复了，代码也比较多，请查看参考原文或者[例子源码](https://github.com/kekxv/kProxyCpp/blob/8c34f17d24e9f425d9b84c1e7611cc0b27c0b434/main.cpp#L295 "例子源码")。

### 前端代码

前端代码比较简单点，读取文件 `Base64` 格式字符串，然后一起传入到后台即可：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>OpenCV合并图片</title>
    <script>
        let data = {};

        function onChange(type) {
            let file = this.files[0];
            let reader = new FileReader();
            reader.onload = function (event) {
                // 图片的base64编码会在这里被打印出来
                data[type] = (event.target.result);
                let img = document.querySelector(`#${type}_img`);
                img.src = data[type];
            };
            reader.readAsDataURL(file)
        }

        function MergePhoto() {
            let httpRequest = new XMLHttpRequest();//第一步：创建需要的对象
            httpRequest.open('POST', '/MergePhoto', true); //第二步：打开连接/***发送json格式文件必须设置请求头 ；如下 - */
            httpRequest.setRequestHeader("Content-type", "application/json");
            httpRequest.send(JSON.stringify(data));//发送请求 将json写入send中
            /**
             * 获取数据后的处理程序
             */
            httpRequest.onreadystatechange = function () {//请求后的回调接口，可将请求成功后要执行的程序写在其中
                if (httpRequest.readyState === 4 && httpRequest.status === 200) {//验证请求是否发送成功
                    let json = httpRequest.responseText;//获取到服务端返回的数据
                    try {
                        json = JSON.parse(json);
                    }catch (e) {

                    }
                    if(json.code===0) {
                        document.querySelector("#MergePhoto").src = json.photo;
                    }else{
                        alert(json.message);
                    }
                }
            };
        }
    </script>
    <style>
        img {
            max-height: calc(100% - 1.5em);
            max-width: 100%;
        }

        img[src=""] {
            opacity: 0;
        }
    </style>
</head>
<body>
<div style="position: fixed;left: 0;right: 0;top:0;bottom: 50%;">
    <div style="padding:0.5em;box-sizing: border-box;border:1px solid #CCCCCC;position: absolute;left: 0;right:  50%;top:0;bottom:0;">
        <label>左边图片：<input type="file" onchange="onChange.call(this,'left')"></label><br/>
        <img id="left_img" src="" alt="左边图片"/>
    </div>
    <div style="padding:0.5em;box-sizing: border-box;border:1px solid #CCCCCC;position: absolute;left:  50%;right: 0;top:0;bottom: 0;">
        <label>右边图片：<input type="file" onchange="onChange.call(this,'right')"></label><br/>
        <img id="right_img" src="" alt="右边图片"/>
    </div>
</div>
<div style="padding:0.5em;box-sizing: border-box;border:1px solid #CCCCCC;position: fixed;left: 0;right: 0;top: 50%;bottom: 0;">
    <button onclick="MergePhoto()">合并</button>
    <br/>
    <img id="MergePhoto" src="" alt="合并之后图片">

</div>
</body>
</html>
```

## 最终效果

![最终效果](/assets/imgs/old/2019/12/201912221453431857520.png "最终效果")

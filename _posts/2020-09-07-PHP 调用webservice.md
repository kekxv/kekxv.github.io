---
layout: post
title: "PHP 调用webservice"
tagline: "PHP 调用webservice"
categories: PHP
author: "kekxv"
---

很多时候，我们需要调用第三方的接口，例如使用 `PHP`调用 `webservice` 接口。

关于 `webservice` 接口，我们可以直接使用`SoapUi`查看或者像对方索要具体地址函数以及参数。

在`PHP`里面调用的话，可以使用 `SoapClient`，一般`PHP`都会开启该扩展，并且相对也比较简单。

第一一个`SoapClient`对象：

```php
// 解决OpenSSL Error问题需要加第二个array参数，具体参考 http://stackoverflow.com/questions/25142227/unable-to-connect-to-wsdl
$client = new SoapClient("webservice地址?wsdl",
        array(
            "stream_context" => stream_context_create(
                [
                    'ssl' => ['verify_peer' => false,'verify_peer_name' => false,]
                ]
            )
        )
    );
```

调用方式可以直接使用箭头(→)方式调用，注意保证函数名以及参数正确:

```php
$parm = ['参数' => "参数值"];// 可多个参数
//SendPersonMMS 为函数名
$result = $client->SendPersonMMS($parm);
//输出结果 
var_dump($result);
```

完整的操作为：（ 参考：[https://www.cnblogs.com/xjnotxj/p/6212143.html](https://www.cnblogs.com/xjnotxj/p/6212143.html) ）

```php
header("content-type:text/html;charset=utf-8");
try {

    //解决OpenSSL Error问题需要加第二个array参数，具体参考 http://stackoverflow.com/questions/25142227/unable-to-connect-to-wsdl
    $client = new SoapClient("http://XXX/webservice/mmsservice.asmx?wsdl",
        array(
            "stream_context" => stream_context_create(
                array(
                    'ssl' => array(
                        'verify_peer' => false,
                        'verify_peer_name' => false,
                    )
                )
            )
        )
    );
    //print_r($client->__getFunctions());
    //print_r($client->__getTypes());

    $parm = array('mobile' => ‘136XXXXXX’, 'mmsid' => 'XXX', 'sToken' => 'XXX');
    $result = $client->SendPersonMMS($parm);
    //print_r($result);

    //将stdclass object的$result转换为array
    $result = get_object_vars($result);  
    //输出结果 
    echo $result["SendPersonMMSResult"];
   

} catch (SOAPFault $e) {
    print $e;
}
```

如果使用`XML`格式，还可以使用 `DOMDocument`：

```php
$books = new DOMDocument();
$WsSubmitReq = $books->createElement("参数");
$WsSubmitReq->appendChild($books->createElement("参数", "值"));
$WsSubmitReq->appendChild($books->createElement("mobile", $PHONE));
$WsSubmitReq->appendChild($books->createElement("content", $SmsMessage));
$books->appendChild($WsSubmitReq);
$parm = array('参数' => $books->saveXML());
```

序列化的内容可保证正确无误，并且会自动转码。



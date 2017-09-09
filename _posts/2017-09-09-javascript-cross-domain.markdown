---
layout: post
title:  "javascript跨域"
date:   2017-09-09 19:18:34 +0800
categories: javascript cross-domain lumen php
---

关于跨域问题的入门文章，推荐阮一峰的这篇《跨域资源共享 CORS 详解》 [http://www.ruanyifeng.com/blog/2016/04/cors.html]()

以下简单粗暴的把过程给到大家。

在HTML下，直接在head中加上
```html
<html>
    <head>
        <meta http-equiv="Access-Control-Allow-Origin" content="*">
    ...
```

在原生PHP下，在响应（response）时，加上这两句即可

```php
<?php
    ...
    header('Access-Control-Allow-Origin:*');
    header('Access-Control-Allow-Methods:GET,POST,PUT,OPTIONS');
    ... 
```

如果想指定某个域名下的javascript才能访问，将*改为具体网址

```php
<?php
    ...
    header('Access-Control-Allow-Origin:https://cbcfrank.github.io'); //这里的地址是举例，可以是其他
    header('Access-Control-Allow-Methods:GET,POST,PUT,OPTIONS');
    ... 
```

如果你想指定某几个域名下的javascript才能访问，你会想怎么做？
将多个网址用逗号隔开吗？那么你浏览器F12，看下console会提示你大概像这样

```js
   ...The 'Access-Control-Allow-Origin' header contains multiple values '网址一, 网址二', but only one is allowed. Origin 'http://localhost' is therefore not allowed access.
```

所以这里要做个小改造

```php
<?php
    ...
    $origin = isset($_SERVER['HTTP_ORIGIN'])? $_SERVER['HTTP_ORIGIN'] : '';
    $allow_origin = array('网址一', '网址二', '网址三');

    if(in_array($origin, $allow_origin)) {
        header('Access-Control-Allow-Origin:'.$origin);
        header('Access-Control-Allow-Methods:GET,POST,PUT,OPTIONS');
    }
    ...    
```

相信你明白了吧。（这段代码片段借鉴了 [http://blog.csdn.net/fdipzone/article/details/46390573]() ）

到这里就讲完了。

题外话，加入域js的交互有自定义头部的话，需要加上

```php
    header('Access-Control-Allow-Headers:X-Custom-ABC');
```

以下附上Lumen框架的header写法供参考。

```php
<?php
    ...
    response(
        '{"msg":"返回的内容"}', 
        200, 
        array(
            'Access-Control-Allow-Origin' => '*', 
            'Access-Control-Allow-Methods' => 'GET,POST,PUT,OPTIONS',
            'Access-Control-Allow-Headers' => 'X-Custom-ABC'
        )
    );
```


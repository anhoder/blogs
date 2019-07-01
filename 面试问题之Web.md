# 面试问题之Web

> 博客内容为[PHP-Interview-QA](https://github.com/colinlet/PHP-Interview-QA)读后笔记

## SEO

* 语义化的HTML代码，符合W3C规范
* 重要内容放在最前
* 重要内容不用js输出
* 少使用iframe
* 非装饰图片加alt
* 添加网站地图

## JavaScript代码放在HTML末尾

* 优先加载HTML及CSS，减少用户使用时的加载时长
* 在JavaScript执行之前，保证HTML全部加载

## 跨域方式

* 使用jsonp进行跨域（只能GET请求），创建回调函数，在服务器返回的数据中调用该函数，并将数据放在函数的参数中
* document.domain + iframe，设置两个页面的基础域名相同（子域不同）
* CORS，服务器返回设置Access-Contorl-Allow-Origin: *
* WebSocket




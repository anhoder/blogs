# PHPMailer部署到Linux服务器上发送邮件出错(⊙o⊙)?

<!-- more -->

<h3>旨在完成：Linux服务器在触发某条件时可以自动发送邮件至管理员，已达到提醒的目的。</h3>

于是花了些时间在macOS上使用PHPMailer类完成发送邮件的功能（主要是写CSS样式，强迫症TvT），然后开开心心的将文件打包上传到服务器。

emmm，先测试一下（嘿，运行完成）

打开邮箱，咋没有呢（漫长的等待....）

emmm，还是没有，唉，又到了找问题的时候了，可是也没报错呀QAQ

<h3>搜索了一下，发现PHPMailer自己保存了错误信息(<span style="color: red;">ErrorInfo</span>)，于是将错误信息输出</h3>

```php
if (!$mail->Send()) {
    echo $mail->ErrorInfo;
    return FALSE;
} else {
    return TRUE;
}
```

保存退出，运行...
果然输出了错误信息：

```php
SMTP connect() failed.
```

压根就没连上SMTP服务器，这可咋整？

<h3>然后又是一顿搜索，发现PHPMailer是可以调试的</h3>

```php
// PHPMailer调试

// 0 = off (for production use)

// 1 = client messages

// 2 = client and server messages

$mail->SMTPDebug = 2;
```

将其加入到代码中，运行，输出：

```php
2018-04-06 11:03:05 Connection: opening to smtp.qq.com:25, timeout=300, options=array ()

2018-04-06 11:03:19 SMTP ERROR: Failed to connect to server: Connection timed out (110)

2018-04-06 11:03:19 SMTP connect() failed.
```

<h3>带着错误继续搜索，发现有人通过<span style="color: red;">禁用IPv6</span>解决这个问题，于是我也尝试着禁用了IPv6，可是效果却并不好，错误依然存在，显然这种方法并不适合我的Linux</h3>

<h3>随后又分别检查了<span style="color: red;">OpenSSL扩展是否开启</span>以及<span style="color: red;">相关函数是否被禁用</span>，但是一切正常</h3>

 
最后在百度搜索 SMTP ERROR: Failed to connect to server: Connection timed out (110)，在Stack Overflow上看到PHPMailer的相关配置信息（<a href="https://stackoverflow.com/questions/34953034/phpmailer-smtp-connect-failed-connection-timed-out-110">传送门</a>），于是开始调试自己代码的配置

<h3>当我添加上以下配置信息后</h3>

```php

$mail->Port = 587;

```

<h3>运行发现，没有错误信息，然后检查接收的邮箱也成功收到了邮件</h3>

<h3>需要注意的是：<span style="color: red;">ssl对应端口465，tls对应端口587</span></h3>



<span style="color: red;">在mac系统下，我是没有配置Port这个选项，但是也可以成功发送邮件，具体原因就不得而知了</span>
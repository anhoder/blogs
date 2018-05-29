# 初探Laravel---round 4 邮件发送

*Mission Start!*

最近想实现发送邮件找回密码的功能，一开始想的是用PHPMailer类进行发送，后来发现Laravel是已经封装了发送邮件的功能的（一时没想到ㄟ( ▔, ▔ )ㄏ）。简单记录一下SMTP发送邮件的实现。

## 一、项目配置
① .env文件中：

```
MAIL_DRIVER=smtp        // 除SMTP驱动外，还有Mailgun驱动、Mandrill驱动、SES驱动
MAIL_HOST=smtp.qq.com   // smtp服务器
MAIL_PORT=465           // 加ssl安全加密的对应465端口，未使用ssl则对应25端口
MAIL_FROM_ADDRESS=123456@qq.com 
MAIL_FROM_NAME=
MAIL_USERNAME=123456@qq.com // 用户名
MAIL_PASSWORD=your_password // 密码，QQ使用授权码
MAIL_ENCRYPTION=ssl         // ssl加密层，不加密为tls
```

② /config/mail.php（当.env未配置某个值时，读取该文件中的默认值）：

```
// env方法用于取.env环境配置的值，第一个参数为键，第二个参数为缺省时默认值

'driver' => env('MAIL_DRIVER', 'smtp'), // MAIL驱动
'host' => env('MAIL_HOST', 'smtp.mailgun.org'),  
'port' => env('MAIL_PORT', 587),        // Mail端口
'encryption' => env('MAIL_ENCRYPTION', 'tls'), // 加密方式
'username' => env('MAIL_USERNAME'),  // 用户名
'password' => env('MAIL_PASSWORD'),  // 密码，QQ使用授权码
'sendmail' => '/usr/sbin/sendmail -bs', // sendmail命令
```

## 二、发送邮件

### 基本功能

在需要使用发送邮件功能的文件：

```php
use Mail;

class TestController extends Controller
{
    public function send_mail()
    {
        $view = 'emails.hello_world';
        $data = ['username' => 'Alan', 'msg' => 'Good morning'];
        $mail_to = '654321@qq.com';
        $subject = '这是邮件主题emmm';
        
        Mail::send($view, $data, function ($msg) use ($mail_to, $suject) {
            $msg->to($mail_to)->subject($subject);
        });
        
        return Mail::failures();
    }
}
```
在上述代码中：
> $view为模板文件的位置（/resources/views/emails/hello_world.blade.php）；
>     
> $data为需要传入到模板文件的数据；
>     
> $mail_to为目标邮箱地址；
>     
> $subject为邮件主题。
>      
> 返回值为Mail的错误集，非空则表示发生错误
>     
> *最好将上述变量作为方法的参数传入*

### 模板文件

存放在/resources/views/下，使用 `directory_name.view_name` 即可定位到目标模板文件 `/resources/views/directory_name/view_name.blade.php` 。在模板文件中，使用 `{{$var_name}}` 获取传入的数据（即上面代码中的变量$data，var_name为$data数组中的键）。例如，使用上述传入的数据：

```php
<html>
    <body>
        <h1>尊敬的{{$username}}：</h1>
        <div>{{$msg}}</div>
    </body>
</html>
```

### 添加附件

需要添加附件，使用  `attach()` 方法，该方法第一个参数为附件的绝对路径，第二个参数通过数组来指定文件显示名和MIME类型，例如：

```php
Mail::send($view, $data, function ($msg) {
    // ......
    $msg->attach($path, ['as' => $file_name, 'mime' => $mime_type]);
    // ......
});
```
### 消息队列
#### 普通队列
考虑到发送邮件会大大增加应用的响应时间，我们可以使用Laravel内置的队列API来实现，如：

```php
Mail::queue($view, $data, function ($msg) {
    // 
});
```
需要在使用时配置队列。

#### 延迟队列
延迟发送邮件，可使用 `later` 方法，如：

```php
Mail::later(10, $view, $data, function ($msg) {
    // 
});
```
#### 推入指定队列

```php
Mail::queueOn('queue_name', $view, $data, function ($msg) {
    //
});

```
或

```php
Mail::laterOn('queue_name', 10, $view, $data, function ($msg) {
    //
});
```

*Mission Complete!*



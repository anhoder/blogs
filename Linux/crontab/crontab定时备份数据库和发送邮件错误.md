# crontab定时备份数据库和发送邮件错误

<!-- more -->

最近想使用linux定期备份数据库并发送邮件提醒

于是编写脚本，手动运行，一切都很完美，于是使用

```sh
crontab -e
```

添加定时任务，但是到了时间却未收到邮件，然后<span style="font-size: 22px; color: red;">查看脚本编辑的文件，发现是空的</span>

于是在crontab文件中加入重定向：

```sh
* * * * * command >> /(路径)/log/backupDB.log 2>&1
```

设置定时，等待，然后发现输出错误为<span style="font-size: 22px; color: red;">未找到mysqldump命令</span>
推测应该是脚本中的路径问题，于是打开脚本文件，在<span style="font-size: 22px; color: green;">命令前加上命令的路径</span>

如：/(路径)/mysqldump -uusername -ppassword --all-databases > /(路径)/a.sql


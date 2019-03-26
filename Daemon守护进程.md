# Daemon守护进程

*Mission Start!*

## 0、进程

* 僵尸进程：**当子进程比父进程先结束，而父进程又没有回收子进程，释放子进程占用的资源，此时子进程将成为一个僵尸进程。**如果父进程一直不处理,该进程将会一直存在,占用系统进程表项。
* 孤儿进程：**在其父进程执行完成或被终止后仍继续运行的一类进程。**这些孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。孤儿进程没有任何危害,只是需要注意自己的代码逻辑即可。

## 1、进程、进程组、会话、控制终端的关系

* 进程：进程属于且仅属于一个进程组；
* 进程组：进程组包含多个进程，进程组号就是进程组长的进程号；
* 会话：会话包含多个进程组，这些进程组可以分为：后台进程组、前台进程组
* 控制终端：控制终端对应一个或多个登录会话（screen命令可以创建会话）

关系如图：

![进程、进程组、会话、控制终端](https://api.alan123.xyz/2019/03/23/5c963c3a9036d.jpg)

## 2、守护进程的创建过程

![守护进程](https://api.alan123.xyz/2019/03/23/5c95a53bc04d5.png)

> ① fork出的子进程会在继承父进程的状态，并往下执行，且子进程属于父进程所在的进程组和会话。<font style="color:red;">**使用posix_setsid函数的作用就是创建一个新的会话，让子进程摆脱父进程所在的进程组和会话的控制**</font>
> 
> ② <font style="color:red;">命令行使用&符号在后台运行程序，只是将该程序放到后台进程组，并没有脱离当前进程组和会话的控制</font>，因此，当用户退出登录时，程序就会结束
> 
> ③ POSIX：portable operating system interface of UNIX


示例代码：


```php
<?php
$pid = pcntl_fork();
if ($pid == -1) {
    die("Fork Error");
} elseif ($pid) {
    // 父进程
    exit();
} else {
    // 子进程
    posix_setsid();
    chdir('/');
    umask(0);
    for ($i = 0; $i < 20; $i++) {
        echo $i, PHP_EOL;
        sleep(1);
    }
}
```

## 3、结语

可以将上述代码优化封装成一个Daemon守护进程类，添加**实现信号量处理（启动、暂停、结束等）、添加执行任务、保存进程id到pid文件等功能**。    
    
示例代码：

```php
class Daemon
{
    private $pid_file;
    private $log_file;
    private $jobs = [];

    public function __construct($log_dir = '/tmp/', $pid_dir = '/var/run/')
    {
        $this->checkCli();
        $this->checkPcntl();
        $this->setLogFile(rtrim($log_dir, '/') . '/' . __CLASS__ . '.log');
        $this->setPidFile(rtrim($pid_dir, '/') . '/' . __CLASS__ . '.pid');
    }

    /**
     * 设置日志文件
     * @param string $log_file 日志文件路径
     */
    private function setLogFile($log_file)
    {
        $this->log_file = $log_file;
    }

    /**
     * 设置pid文件
     * @param string $pid_file pid文件路径
     */
    private function setPidFile($pid_file)
    {
        $this->pid_file = $pid_file;
    }

    /**
     * 检验是否安装pcntl扩展
     * @return void 
     */
    private function checkPcntl()
    {
        if (!function_exists('pcntl_fork')) {
            throw new Exception("Extension pcntl not exists");
        }
    }

    /**
     * 检查运行环境是否是命令行
     * @return void 
     */
    private function checkCli()
    {
        if (php_sapi_name() != 'cli') {
            throw new Exception("Current running environment is not cli");
        }
    }

    /**
     * 添加任务
     * @param array $job 任务数组
     */
    public function addJob($job)
    {
        if (!isset($job['callback']) || empty($job['callback'])) {
            throw new Exception("Job need callback");
        }
        array_push($this->jobs, $job);
    }
    
    /**
     * 创建守护进程
     *
     * @return void
     */
    private function daemonize()
    {
        $pid = pcntl_fork();
        if ($pid < 0) {
            throw new Exception('Fork son process error');
        } elseif ($pid > 0) {
            exit();
        }
        
        if (posix_setsid() == -1) {
            throw new Exception('Cannot detach session');
        }
        chdir('/');
        umask(0);
        if (!$fp = fopen($this->pid_file, 'w')) {
            throw new Exception('Cannot create pid file');
        }
        fwrite($fp, posix_getpid());
        if (!empty($this->jobs)) {
            foreach ($this->jobs as $job) {
                if (isset($job['argv'])) {
                    call_user_func($job['callback'], $job['argv']);
                } else {
                    call_user_func($job['callback']);
                }
            }
        }
    }
    
    /**
     * 启动守护进程
     *
     * @return void
     */
    private function start()
    {
        if ($this->getPid()) {
            $this->tips('Process is running...');
            return;
        }
        $this->tips('Start success.');
        $this->daemonize();
    }

    /**
     * 停止进程
     *
     * @return void
     */
    private function stop()
    {
        if ($pid = $this->getPid()) {
            posix_kill($pid, SIGTERM);
            unlink($this->pid_file);
            $this->tips('Stop success.');
        } else {
            $this->tips('Process not running.');
        }
    }

    /**
     * 获取状态
     *
     * @return void
     */
    private function status()
    {
        if ($this->getPid()) {
            $this->tips('Process is running...');
        } else {
            $this->tips('Process is stopped.');
        }
    }

    /**
     * 输出提示信息
     *
     * @param string $msg 提示内容
     * @return void
     */
    private function tips($msg)
    {
        printf("%s: %s\n", date('Y-m-d H:i:s'), $msg);
    }

    /**
     * 获取守护进程pid
     *
     * @return void
     */
    private function getPid()
    {
        if (!file_exists($this->pid_file)) {
            return 0;
        }
        $pid = intval(file_get_contents($this->pid_file));
        if (posix_kill($pid, SIG_DFL)) {
            return $pid;
        } else {
            unlink($this->pid_file);
            return 0;
        }
    }

    /**
     * 运行
     *
     * @return void
     */
    public function run($argv)
    {
        if (count($argv) < 2) {
            $this->tips('Params: stop | start | status');
            return;
        }
        switch ($argv[1]) {
            case 'start':
                $this->start();
                break;
            case 'stop':
                $this->stop();
                break;
            case 'status':
                $this->status();
                break;
            default:
                $this->tips('Params: stop | start | status');
                break;
        }
    }
}


// 测试
$daemon = new Daemon('/Users/Alan_Albert/Desktop/Daemon', '/Users/Alan_Albert/Desktop/Daemon');
$daemon->addJob([
    'callback' => function () {
        while (1) {
            echo "true", PHP_EOL;
            sleep(1);
        }
    }
]);
$params = $argv;
$daemon->run($argv);
```

*Mission Complete!*
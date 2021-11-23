### 基本介绍
supervisord是使用python编写的进程监控程序
### 缺陷
   1. 不能支持后台程序。因为supervisord启动的时候会将进程由前台转到后台，如果程序本身就是后台程序则会失败
   2. 管理的程序必须要由supervisord启动，visord实际上是通过创建子进程的方式创建监控的进程。所以对于不由supervisord启动的进程无法监控。
### 优点
   1. 不通于pidfile形式，采用子进程方式方便的获取监控进程的上下文，当监控进程退出时由操作系统向supervisord发送信号，效率更高。

### 常见命令
```
   supervisorctl stop xxx/ALL        停止进程
   supervisorctl restart xxx/ALL     重启进程
   supervisorctl                     列出当前监控的进程
   supervisorctl update              更新配置文件
   supervisorctl reload              重新加载配置文件
```
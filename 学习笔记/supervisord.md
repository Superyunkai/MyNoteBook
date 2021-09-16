### 基本介绍
supervisord是使用python编写的进程监控程序
### 缺陷
   1. 不能支持后台程序。
   2. 管理的程序必须要由supervisord启动
### 优点
   1. 不通于pidfile形式，采用子进程方式方便的获取监控进程的上下文，当监控进程退出时由操作系统向supervisord发送信号，效率更高。

[TOC]
## 查看库文件的符号表
+ 对于静态库使用命令： `nm -n -C -A xxxx.a`
+ 对于动态库使用命令： `nm -n -C -A -D xxx.so` 
## 设置系统时间
 date -s "20091112 18:30:50" &&hwclock --systohc
 tips: clock 是hwclock的软连接，-w参数已经不再使用

## 查看文件大小
stat 命令


## 查看库是否是64位库
``` shell
    #对于动态库
    file xxx.so
    
    #对于静态库
    objdump -a xxx.a
```

## awk专题
  awk 使用 | 分隔
  位运算命令 lshift(val, count)、rshift(val, count)、compl(val)、and(v1, v2)、or(v1, v2)、xor(v1, v2)
## lsof 命令

+ 命令格式：

  lsof [参数] [文件]
+ 功能：
  用于查看进程打开的文件，打开文件的进程，进程打开的端口。找回/恢复删除的文件
  
  -a 列出打开文件存在的进程
  -c<进程名> 列出指定进程所打开的文件
  -g  列出GID号进程详情
  -d<文件号> 列出占用该文件号的进程
  +d<目录>  列出目录下被打开的文件
  +D<目录>  递归列出目录下被打开的文件
  -n<目录>  列出使用NFS的文件
  -i<条件>  列出符合条件的进程。（4、6、协议、:端口、 @ip ）
  -p<进程号> 列出指定进程号所打开的文件
  -u  列出UID号进程详情
  -h 显示帮助信息
  -v 显示版本信息





## 统计单词在文件出现的次数
grep -o '字符串' file |wc -l

## 检查端口是否畅通
nc [-hlnruz][-g<网关...>][-G<指向器数目>][-i<延迟秒数>][-o<输出文件>][-p<通信端口>][-s<来源位址>][-v...][-w<超时秒数>][主机名称][通信端口...]

-g<网关> 设置路由器跃程通信网关，最多可设置8个。
-G<指向器数目> 设置来源路由指向器，其数值为4的倍数。
-h 在线帮助。
-i<延迟秒数> 设置时间间隔，以便传送信息及扫描通信端口。
-l 使用监听模式，管控传入的资料。
-n 直接使用IP地址，而不通过域名服务器。
-o<输出文件> 指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存。
-p<通信端口> 设置本地主机使用的通信端口。
-r 乱数指定本地与远端主机的通信端口。
-s<来源位址> 设置本地主机送出数据包的IP地址。
-u 使用UDP传输协议。
-v 显示指令执行过程。
-w<超时秒数> 设置等待连线的时间。
-z 使用0输入/输出模式，只在扫描通信端口时使用


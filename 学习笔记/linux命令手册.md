
[TOC]
#### 1.查看库文件的符号表
+ 对于静态库使用命令： `nm -n -C -A xxxx.a`
+ 对于动态库使用命令： `nm -n -C -A -D xxx.so` 
### 设置系统时间
 date -s "20091112 18:30:50" &&hwclock --systohc
 tips: clock 是hwclock的软连接，-w参数已经不再使用

### 查看文件大小
stat 命令


#### 查看库是否是64位库
``` shell
    #对于动态库
    file xxx.so
    
    #对于静态库
    objdump -a xxx.a
```
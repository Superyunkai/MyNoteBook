[TOC]
### 切换堆栈
frame id
up down

### 查看堆栈信息
bt 

### 保存上一次记录的断点
save 命令
save breakpoints   ---- Save Current breakpoint definiation as a script 

save gdb-index     ---- Save a gdb-index file 

save tracepoint    ---- Save current tracepoint  definitions as a script 


### 断点(普通断点、观察断点、捕捉断点)
#### 查看已经定义的断点
info breakpoints [n]

#### 删除断点
delete n

#### 禁用/启用断点
disable/enable [breakpoints] [num ...]  ----- breakpoitns  参数可有可无， num参数为指定的一系列断点编号，如果不加num则默认关闭当前所有断点

#### 普通断点
break命令 简写b 
break hiredis.c:100 
break 函数名 
break  ... if cond   ------ cond 代表一个条件表达式，只有cond为TRUE时才会发生中断

#### 观察断点
watch expr
watch expr if cond  ------ cond 代表一个条件表达式，只有cond为TRUE时才会发生中断

#### 捕捉断点
catch event 
event事件类型有 

| event                             | 含义                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| throw [exception]                 | 当程序中抛出 exception 指定类型异常时，程序停止执行。如果不指定异常类型（即省略 exception），则表示只要程序发生异常，程序就停止执行。 |
| catch [exception]                 | 当程序中捕获到 exception 异常时，程序停止执行。exception 参数也可以省略，表示无论程序中捕获到哪种异常，程序都暂停执行。 |
| load [regexp]<br/>unload [regexp] | 其中，regexp 表示目标动态库的名称，load 命令表示当 regexp 动态库加载时程序停止执行；unload 命令表示当 regexp 动态库被卸载时，程序暂停执行。regexp 参数也可以省略，此时只要程序中某一动态库被加载或卸载，程序就会暂停执行。 |

其它写法[[Set Catchpoints (Debugging with GDB) (sourceware.org)](https://sourceware.org/gdb/current/onlinedocs/gdb/Set-Catchpoints.html#Set-Catchpoints)]

注意，当前 GDB 调试器对监控 C++ 程序中异常的支持还有待完善，使用 catch 命令时，有以下几点需要说明：
对于使用 catch 监控指定的 event 事件，其匹配过程需要借助 libstdc++ 库中的一些 SDT 探针，而这些探针最早出现在 GCC 4.8 版本中。也就是说，想使用 catch 监控指定类型的 event 事件，系统中 GCC 编译器的版本最低为 4.8，但即便如此，catch 命令是否能正常发挥作用，还可能受到系统中其它因素的影响。
当 catch 命令捕获到指定的 event 事件时，程序暂停执行的位置往往位于某个系统库（例如 libstdc++）中。这种情况下，通过执行 up 命令，即可返回发生 event 事件的源代码处。
catch 无法捕获以交互方式引发的异常。


#### condition 命令
condition命令主要用于为现有断点增加/修改条件判断。例如catch断点无法直接加条件判断，需要通过condition添加。
condition bnum expr

#### 忽视断点
ignore bnum n   ----忽视某个断点n次


### trace 追踪点
trace  ----用法同break
trace 会在打点的地方短暂中断，收集信息然后继续向下执行。

### 打印内存中的内容
gdb使用x命令打印内存中的值，命令格式为`x/nfu addr`
```
    n ----打印的单元数
    f ----打印的格式
          x 按16进制打印
          d 按10进制格式显示变量
          u 按16进制显示无符号整型
          o 按8进制显示
          t 按二进制格式显示
          a 按16进制大写显示
          c 按字符格式显示
          f 按浮点数格式显示 
```
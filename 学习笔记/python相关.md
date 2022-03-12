<!--
 * @Author: your name
 * @Date: 2022-01-17 09:09:27
 * @LastEditTime: 2022-01-17 10:27:47
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \MyNoteBook\学习笔记\python相关.md
-->

[TOC]



#### python 连接mysql数据库(mysqlclient)

1.  安装mysql-client 
   ```
        yum install mysql
        yum install MySQL-python
   ```

2. 引入
   `import MySQLdb`

3. 基本使用
    + 创建与数据库的连接
        `conn = MySQLdb.connect(host = 'localhost', port = 3306 , user = 'root', password = '123456', db = 'test', charset='utf8')`
    + 获取数据库索引 
        `cur = conn.cursor()`
    + 插入数据
        ``` python
            # 执行一条语句
            sql = "insert into student values(%s, %s, %s, %s)"
            cur.execute(sql, ('3', 'Huhu', '2 year ago', '7'))
            cur.close()
            conn.commit()

            # 执行多条语句, 返回值为受影响的行数
            cur.execute(sql,[(...)),(...)])
        ```
    + 查询数据
        ``` python
            #返回查询出的数据条数
            res_count = cur.execute("select * form students")
            #获取一条数据，同时游标移位
            cur.fetchone()
            #移动游标
            cur.scroll(0. 'absolute')
            #获取n条
            res = cur.fetchmany(n)
            for line in res:
                #这步可能无法打印出中文，这是由于line[i]为二进制流，python的print函数不会自动对其编码，使用的标准输出流中的编码集(ASCII)
                #可以设置1.export LC_ALL="en_US.utf8" 2. import sys ,codecs / sys.stdoput = codecs.getwriter("utf-8")(sys.stdoput.detach)
                print line[i
            cur.close()
            conn.commit()
            cnn.close()
        ```

#### python 查询etcd数据库
    1. 
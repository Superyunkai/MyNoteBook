#### 1.数字比较大小
常用的命令：
        -gt     大于
        -lt     小于
        -eq     等于
        -ne     不等于
        -ge     大于等于
        -le     小于等于

示例：
``` shell
    if [ $1 -gt $2 ]
    then echo ""
    else echo ""
    fi
```

#### 2.shell脚本字符串判空

#### 3.统计文本中字符重复次数
假设有一个文本文件，每行为一个字符串，统计每行的内容在文件中重复的次数：
```shell
# 统计文件中每行出现的次数，并计算出每行出现次数的百分比占比
awk '{a[$0]++} END{for(i in a){printf("%s %d %.2f%%\n", i, a[i], a[i]/NR*100)}}' "$file_path" | sort -nr -k2
``` 

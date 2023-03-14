#### 1.关联任务地址：
       http://172.20.200.191:8080/browse/SJCGZZ-15167
#### 2. 进度更新：
几个面板下的sql语句：
1. b2c-sjzz-sync_selfstock
    + 生产者每秒 产生的消息数：
    ```sql
        sum(rate(kafka_topic_partition_current_offset{topic=~"$topic",job="sjzz-kafka-exporter"}[2m]))by(topic)
    ```
    
 
#### 3.存在的问题

#### 4.ps

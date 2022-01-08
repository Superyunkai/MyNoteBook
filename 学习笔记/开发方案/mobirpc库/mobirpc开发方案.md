## mobirpc库支持行情推送开发方案

【需求标题】：mobirpc库支持行情推送  
【需求方】：手机主站  
【开发者】：王云凯  
【方案Review】：刘智强  
【JIRA】：[mobirpc库支持行情推送]:http://172.20.200.191:8080/browse/SJCGZZ-10694  
【概率发布】：不支持  
【需求细分】：  
+ 1、mobirpc库增加新对外接口，支持回调函数的注册。  
+ 2、自动化测试python代码中增加接受推送的回调函数，在回调函数中检查推送频率。
+ 3、自动化测试python代码增加对应推送检查项的检查函数。
+ 4、自动化测试用例更新对于早盘推送检查和全天推送监控的用例。

【方案说明】  
1. mobirpc库在原有对外接口基础上提供一个新的对外接口，接口最后一个参数为外部回调函数的指针。函数内部在发送推送类型请求后，每次接收到推送数据时调用此回调函数，将数据传递给外部。
2. 自动化测试python部分设计一个回调函数，并通过ctype包中CFUNCTYPE特性注册回调函数，传递给mobirpc库新的接口。回调函数生命周期必须大于rpc库调用的周期。
3. 回调函数中检查推送频率，以推送间隔1分钟为例，回调函数记录上次受到推送的时间，与本次接受推送的时间做比较，不符合条件则返回错误。
4. 检查用例分两种情况，一种早盘检查：检查4～5次推送间的间隔。一种线上监控：持续检查推送结果，异常时发出告警。

【业务影响面】  
手机主站mobirpc库请求的性能、自动化测试用例早盘检查执行时间


[具体修改]
+ PushManager类
    1. 全局变量g_PushDataQueue,队列Node：{PushData, timeStamp}。g_Result标志是否出现错误。g_Count:检查推送次数。g_PushInterval：推送时间间隔
    2. 全局回调函数void CallBack(char* pszdata),负责处理回调的数据，加入g_PushDataQueue。并计算两次推送数据的时间间隔，如果间隔大于预期值，置g_Result为False。如果为监控模式，则告警后重置g_Result。（对于全局变量的修改要确保线程安全)
    3. 提供给其它模块访问、设置全局变量的入口
    4. 数据正确性校验（外部传入，回调函数形式）？
+  HqServer引入PushManager类
    1. 调用推送请求,设置检查的模式，设置g_PushInterval, g_Count
    2. 等待回调函数设置g_PushDataQueue,g_Result
+ 推送数据回调函数const char* Callback( char *pszdata)




mysql_conf
MYSQL= os.environ.get('mysql_address')
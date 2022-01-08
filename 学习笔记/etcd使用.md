##### etcdop 工具
[Git仓库地址](https://github.com/netremo/etcdop)

导出数据：
```
./etcdop -url http://address:port -out filename.yaml 
```

导入数据：
```
./etcdop -url http://address:port -in filename.yaml 
```


#### etcd 开启身份验证
etcd开启身份验证需要首先有root用户。etcd基于role 和 user两种方式隔离权限。
```
    etcdctl --endpoints=${ENDPOINTS} role add root
    etcdctl --endpoints=${ENDPOINTS} role get root
    etcdctl --endpoints=${ENDPOINTS} user add root
    etcdctl --endpoints=${ENDPOINTS} user grant-role root root
    etcdctl --endpoints=${ENDPOINTS} role add rolename
    etcdctl --endpoints=${ENDPOINTS} user add username
    etcdctl --endpoints=${ENDPOINTS} user grant-role username rolename 
    etcdctl --endpoints=${ENDPOINTS} auth enable

```

##### etcd一些环境变量配置
1. etcd的接口分为v2和v3，两者并不兼容，使用v3接口时需要注入环境变量 `ETCDCTL_API=3`
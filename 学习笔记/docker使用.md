[TOC]
### 创建mobiroute容器
docker run  -d -e SJZZ_ENV_DOCKER=true --restart=always --privileged --net=host --name=mobiroute_test --cap-add=ALL -v /root/mobile/mobiroute/resources/inicfg:/root/mobile/mobiroute/resources/inicfg hub.hexin.cn:9082/sjzz/mobiroute:1.0.0-20211118103618.liuzhiqiang.835ec7f5.120.release-20211028-SJCGZZ-11297 -l 9771 -n 1 -r default_10

### 创建mobiauth容器
docker run  -d -e SJZZ_ENV_DOCKER=true --restart=always --privileged --net=host --name=mobiauth_64 --cap-add=ALL -v /root/mobile/mobiauth/resources/authkey:/root/mobile/filesync/authkey -v /root/mobile/mobiauth/resources/inicfg:/root/mobile/filesync/inicfg -v /root/mobile/mobiauth/resources/ip2region:/root/mobile/filesync/ip2region -v /root/mobile/mobiauth/resources/jsoncfg:/root/mobile/filesync/jsoncfg -v /root/mobile/mobiauth/resources/usersfile:/root/mobile/filesync/usersfile -v /root/mobile/mobiauth/resources/yybset:/root/mobile/filesync/yybset hub.hexin.cn:9082/sjzz/mobiauth:1.0.0-20211108150250.23574232 -l 9541 -n 1

### 查看容器的挂载目录
docker inspect container_name/container_id |grep Mounts -A 50


### 容器权限问题
有时使用docker ps 等命令的时候会出现
**Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/json: dial unix /var/run/docker.sock: connect: permission denied
**
这通常是由于当前用户没有权限解决的办法有以下几种
1. `sudo setfacl --modify user:<user name or ID>:rw /var/run/docker.sock` 这个方法比usermod 或 chown 更加安全且不需要重启机器，需要注意的是如果用户名仅仅存在于docker内部则需要提供ID
2. 
   ```
    sudo usermod -aG docker $USER
    sudo rebot
    ```
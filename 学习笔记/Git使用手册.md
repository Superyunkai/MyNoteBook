 [TOC] 
## Git工具使用技巧
#### 1、给文件增加可执行权限
```
    git update-index --chmod +x xxx.sh
```
#### 2、压缩commit
```
    git commit -i^x (x为要压缩的版本号，或版本数)
```
#### 3、修改历史commit信息
+ `git rebase -i commitId` , 将需要修改的commit改为edit。如果需要修改第一次提交，可以用 `git rebase -i --root` 
+ 使用`git commit --amend` 命令修改commit的message。
+ `GIT_COMMITTER_DATE="T" git commit --amend --date="T" ` 修改commit的时间, 这里没法用xagrs，批量修改非常难受。需要通过脚本的方式实现。 


#### 4、Git仓库迁移
```
    git remote add origin2 master //新建一个origin远程连接，不要与现有的重名
    git remote set-url origin2 xxxxxx[迁移目的仓库的git地址]
    git push origin2
```
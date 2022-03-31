 [TOC] 
## Git工具使用技巧
#### 给文件增加可执行权限
```
    git update-index --chmod +x xxx.sh
```
#### 压缩commit
```
    git commit -i^x (x为要压缩的版本号，或版本数)
```
#### 修改历史commit信息
+ `git rebase -i commitId` , 将需要修改的commit改为edit。如果需要修改第一次提交，可以用 `git rebase -i --root` 
+ 使用`git commit --amend` 命令修改commit的message。
+ `GIT_COMMITTER_DATE="T" git commit --amend --date="T" ` 修改commit的时间, 这里没法用xagrs，批量修改非常难受。需要通过脚本的方式实现。 


#### 4、Git仓库迁移
```
    git remote add origin2 master //新建一个origin远程连接，不要与现有的重名
    git remote set-url origin2 xxxxxx[迁移目的仓库的git地址]
    git push origin2
```

#### Git删除本地分支
```
    git branch - && git branch -a          ----查看本地和远程的分支
    git branch -d <branch-name>            ----会检查是否处于merge状态
    git branch -D <branch-name>            ----强制删除
```
#### gitignore语法
+ 用于忽视某些文件，语法类似于简化的正则表达式
+ 所有空行或者以 ＃ 开头的行都会被 Git 忽略。
+ 可以使用标准的 glob 模式匹配。
+ 匹配模式可以以（/）开头防止递归。
+ 匹配模式可以以（/）结尾指定目录。
+ 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。
+ 所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。
+ 星号（*）匹配零个或多个任意字符；
+ [abc]匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；
+ 问号（?）只匹配一个任意字符；
+ 如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。
+ 使用两个星号（**) 表示匹配任意中间目录，比如a/**/z可以匹配 a/z, a/b/z 或 a/b/c/z等。
  
  对于在添加gitignore前已经提交过的工程
  ```
    git rm -r --cache .
    git add .
    git commit -m '....'
    git push
  ```

#### 设置git代理
//取消http代理
git config --global --unset http.proxy
//取消https代理 
git config --global --unset https.proxy

#### git cherry-pick 

+ 作用： apply some changes introduced by some exist commit
+ 用法: git cherry-pick commitid1 commitid2
+ Note: git cherry-pick 会在应用的分支上产生一个新的commitid，不会使用原有的commitid  


#### 让vscode记住用户名和密码
git设置好git的用户名密码
git config --global credential.helper store
# 解决子模块冲突
假如当前在 branchA，在执行 git merge branchB 时，可能会遇到子仓库文件冲突

可以尝试一下命令：
'''
git update-index --cacheinfo 0160000, <commit-hash>, "<子仓库path>"
 
//例子
git update-index --cacheinfo 0160000,533da4ea00703f4ad6d5518e1ce81d20261c40c0,module-common 
'''
其中 <commit-hash> 改为 branchA 对应子仓库的 commit 哈希值

其中 <子仓库path> 为在 .gitmodules 配置的子仓库 path
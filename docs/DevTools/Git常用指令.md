# Git常用指令

# 常用指令
- `git clone 地址`
- `git checkout -b development` 创建本地分支并切换到这个分支
- `git checkout --track origin/test-dev` 从远程拉分支
- `git push origin development` 创建远程分支
- `git branch -u origin/luozhengshun` 建立本地远程联系
- `git branch -a` 查看远程分支
- `git branch` 查看本地分支
- `git pull` 更新本地库
- `git add .`添加到本地库
- `git commit -m '提交属性' `提交达到本地库
- `git push` 提交远程库
- `git status` 查看状态
- `git diff xxx`查看更改
- `git log` 查看历史
- `git checkout 分支名`切换分支
- `git branch -d 分支名`删除本地分支
- `git push origin --delete 分支名`删除远程分支
- `git checkout master` 切换到Master分支
- `git merge —no-ff development` 对Development分支进行合并
- `git remote` 列出所有远程主机
- `git remote update origin --prune` 更新远程主机origin 整理分支
- `git branch -r` 列出远程分支
- `git branch -vv` 查看本地分支和远程分支对应关系
- `git checkout -b gpf origin/gpf` 新建本地分支gpf与远程gpf分支相关联
- `git reset --soft HEAD~1` 撤销上一次conmmit，1代表上最近1次，若想撤销最近2次则改为2

# 总结

- 一般我们就用`git push --set-upstream origin branch_name`来在远程创建一个与本地branch_name同名的分支并跟踪；
- 利用`git checkout --track origin/branch_name`来在本地创建一个与branch_name同名分支跟踪远程分支-
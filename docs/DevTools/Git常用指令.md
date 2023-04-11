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

# 使用技巧

## 查看最近提交

```
git show
或
git log -n1 -p
```

## 重写commit信息（未push）

```
git commit --ament --only -m 'xxxxxxx'
也可以通过git commit --ament --only打开编辑器编辑
```

## 从commit中移除一个文件

```
git checkout HEAD^ 文件
git add -A
git commit --amend
```

## 删除最后一次commit

### 未push

```
git reset --soft HEAD@{1} 
```

### 已push

撤销提交并强制覆盖远程分支。

:warning: 谨慎使用！！！在此之前要保证其他人不会使用该分支。

```
git reset HEAD^ --hard
git push -f [remote] [branch]
```

安全的方法：创建一个新的提交用于撤销前一个提交的所有变化

```
git revert SHAofBadCommit
```

## 删除任意commit

:warning: 谨慎使用！！！在此之前要保证其他人不会使用该分支。

```
git rebase --onto SHA1_OF_BAD_COMMIT^ SHA1_OF_BAD_COMMIT  
git push -f [remote] [branch]
```

## 撤销hard reset

```
查看提交列表
git reflog
找到想要回到的提交
git reset --hard SHA值
```

## 选择暂存文件的一部分

```
git add -p filename.x
随后在交互界面中使用s选项分隔提交
```

## 将未暂存的内容提交到新分支

```
git checkout -b 新分支
```

## 将未暂存的内容提交到其他已有分支

```
git stash  
git checkout my-branch  
git stash pop
```

## 丢弃本地未提交的变化

```
# one commit  
git reset --hard HEAD^
# two commits  
git reset --hard HEAD^^
# four commits  
git reset --hard HEAD~4
# or  
git checkout -f
```

重置某个文件

```
git reset filename
```

## 丢弃本地未暂存的内容

签出不需要的内容

```
git checkout -p 
```

使用stash保留需要的内容

```
git stash -p  
# Select all of the snippets you want to save  
git reset --hard  
git stash pop  
```

stash 不需要的部分

```
git stash -p  
# Select all of the snippets you don't want to save  
git stash drop  
```

## 重置本地分支与远程保持一致

首先要确保本地没有push到远程

```
git reset --hard origin/分支
```

## 撤销rebasing和merging

```
git reset --hard ORIG_HEAD
```

## 安全合并分支

```text
git merge --no-ff --no-commit 分支
```

`--no-commit` 执行合并(merge)但不自动提交, 给用户在做提交前检查和修改的机会。

`no-ff` 会为特性分支(feature branch)的存在过留下证据, 保持项目历史一致。

## 将一个分支合并成一个提交

```
git merge --squash 分支
```

## 检查分支上所有的commit是否都已经合并

```
git log --graph --left-right --cherry-pick --oneline HEAD...feature/120-on-scroll  
```

## 将历史提交创建新分支

```
git checkout -b 新分支名 提交版本号
如：git checkout -b new-branc f5e86fbf455960c40e759c48bb0675b54d808632
# 查看当前分支
git branch
# 推送分支到远程
git push origin 本地新分支名:远程新分支名
```


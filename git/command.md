## 常用git命令

## 提交流程

```javascript
// 获取状态
git status

// 添加到暂存区
git add 

// 提交
git commit -m '内容'

// 修改分支名称
git branch -m main

//删除分支名称
git branch -d dev

// 如遇到问题修改不想产生新的提交记录
git rebase -i head~N / commit Id

// 修改提交的标题
git commit -amend

// 拉取远程最新分支(基于远程分支变基)
git pull origin dev --rebase

// 若想基于本地分支变基
git rebase dev 

// 开发新需求时 突然 有bug提给你 但是你还没写完 你可以先存储起来
git stash save '缓存简单的名称'
//然后解决完bug之后 切回这个分支 获取你存储的代码
1.首先获取stash 目录,执行`git stash list `获取Id
2.执行`git statsh apply id`

// 提交了某条命令后想要撤回 
git reset head^N / commitID --sort // 不会删除提交的内容
git reset head^N / commitId --hard // 删除提交的内容

```



## 拉取及推送

```javascript
// 设置远程git仓库地址
git remote add origin **.git

//修改远程仓库地址
git remote set-url origin **.git

// 上传到远程仓库相应分支(本地仓库将对应该远程仓库，只有第一次push需要设置）
git push --set-upstream origin main

// 获取远程分支
git branch --remote

// 创建本地分支并同步了远程分支
git checkout -b '本地你想设置的分支名称' origin/分支名称

```

## 总结

```
// 添加到暂存区
git add 
// 提交
git commit -m '内容'
// 获取远程分支
git branch --remote
// 获取所有分支
git branch --all
// 切换远程分支
git checkout -b '本地分支名称'  origin/远程分支名称
// 获取当前分支远程最新代码
git fetch origin
// 拉取某一条commit
git cherry-pick 8838232
// 交互式变基
git rebase -i head~N
// 重置提交 commit 返回暂存区
git reset head^N --sort
// 重置提交 commit 删除
git reset head^N --hard
// 变基
git rebase dev
// 基于 远程分支变基(可以获取远程最新的代码)
git pull origin dev --rebase
// 缓存
git stash save '本次缓存名称'
// 获取缓存列表
git stash list
// 恢复某次暂存 会删除缓存
git stash pop 'stashId'
// 恢复缓存 不会删除缓存
git stash apply 'stashId'
//强制推送(变基后常用)
git push -f
// 清空暂存区
git checkout .
git clean -d
// 删除本地分支
git branch -d dev
// 获取远程仓库地址
git remote -v
// 设置远程仓库地址
git remote add origin **.git
// 修改远程仓库地址
git remote set-url  origin **.git
// 指定上传到远程的分支名称
git push --set-upstream origin main
// 改名 
git branch -m main
// 修改提交信息
git commit --amend
```


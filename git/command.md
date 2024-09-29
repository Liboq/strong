## 常用git命令

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
// 修改远程仓库地址
git remote set-url  origin **.git
```


# rebase实践

> 公司将践行 git rebase ，不太熟悉 所以有必要实践一下
>
> 公司的git管理是这样的
>
> main分支 :生产
>
> dev：( gitlab 成员(无权限) 推送分支 )
>
> test: 指向需发测试分支（不固定）(成员无权限)
>
> hotfix:线上问题 修复 分支
>
> 每次修改提交前都需要 rebase origin/dev 提前解决冲突 不把冲突给到合并分支的同事

## git 工作流

> 场景：同事 a 负责 项目的 某个需求  同事 b 负责 解决项目的 某个bug 同时进行开发，同事a先完成, 同事b后完成，如何进行git操作

首先 我在 `github`上新建一个项目 主分支 `main`

然后 创建一个新的分支 `dev` 

从 dev 又拉取一个分支 `feat` ,`feat`分支进行修改提交

>修改提交流程
>
>```
>git add something
>git commit -m "something"
>git reabse origin/dev
>---若出现冲突---
>处理冲突
>git add .
>git rebase --continue
>```
>
>

继续切换到 `dev` 拉取一个新分支 `fix`,随后 进行 修改提交

随后 在 github上 对  fix 进行 pull request   然后 review 进行 rebase merge

![image-20240523113627098](https://cdn.liboqiao.top/markdown/image-20240523113627098.png)

然后 feat分支进行同样的操作

从git graph上看到

![image-20240523114738928](https://cdn.liboqiao.top/markdown/image-20240523114738928.png)

`dev`分支是在一条线上的，没有产生`merge`多出来的一条`commit`,完成

流程图如下

![rebase](https://cdn.liboqiao.top/markdown/rebase.drawio.png)

## 测试

固定test分支 test 分支 不做修改 只改指向 ，开发人员 每次都需从dev拉取

合并 dev 分支 后，若需要更新到测试， 则 改变test分支指向

 ```
 git checkour test 
 git reset --hard origin/dev
 git push -u origin test
 ```

## 测试完成 推送正式

把 dev 分支 合到 main 分支

## 优点

merge：

rebase

📎 参考文章

`https://blog.csdn.net/XJ5210224/article/details/124705868`


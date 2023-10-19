查看状态

> `git status`

选中提交的文件

> `git add fileName....`

撤回提交的文件

>`git reset <fileName>`

提交更改

> `git commit -m "<log>"`

推送至远程服务器

> `git push <remote> <branch>`

增加远程服务器

> `git remote add <remote Name> <remote URL>`

查看remote version

> `git remote -v`

查看分支

> `git branch`

从远程克隆代码

> `git clone <远程地址> <下载路径>`

初始化一个新的仓库

> `git init`

设置UserName & Email

> `git config --global user.name "<用户名>"`
>
> `git config --global user.email "<Email>"`

回退版本
> `git reset HEAD^`  回退到上一个版本

> `git restore <fileName>` 将filename文件恢复到未修改状态

> `git checkout -b <branch name> <SHA1>` 在提交版本`SHA1`处创建分支

删除分支
> `git branch -d <branch name>`

新建分支
> `git checkout -b <branchName>`

切换分支
> `git checkout <branchname>`

合并指定分支到当前分支
> `git merge <branchName>`

中文乱码配置

> `git config core.quotepath false  --global`

使用全局代理
> `git config --global http.proxy "http://ip:port"`  
> `git config --global https.proxy "https://ip:port"`

移除代理
> `git config --global --unset http.proxy`  
> `git config --global --unset https.proxy`

撤销回退
> `git reflog`  查找版本log
> `git reset <commit id>`

拉取远程的指定分支到本地
> `git checkout -b test <name of remote>/test`

合并多个提交
> `git rebase -i HEAD~n`
# Git Note

## 本地分支关联远程的情况

`git branch -vv`

## 关联远程分支

`git branch -u origin/yourbranch`

或者 push 时加上 -u

`git push origin/yourbranch -u`

## 列出本地和远程分支

`git branch -a`

## 创建并切换到远程分支

`git checkout -b <branch-name> orignin/<branch-name>`

## 删除本地分支

`git branch -d <local-branch-name>`

## 删除远程分支

`git push origin --delete <remote-branch-name>`

## 重命名本地分支

`git branch -m <new-branch-name>`

## 本地创建标签

`git tag <version-number>`

## 推送标签到远程

`git push origin --tags`

## 切回到某个标签

`git checkout -b <branch-name> <tag-name>`

## 放弃工作区修改

`git checkout <file-name>`

## 回到某个commit并重新添加一个commit

`git revert <commit-id>`

## 回到某个commit并删除之后的commit

`git reset <commit-id>` 保留工作区

`git reset --hard <commit-id>` 不保留工作区和暂存区

## 修改上个commit的描述

`git commit --amend`

## 查看某段代码是谁写的

`git blame <file-name>` （AS 中右键 Copy Path 可以复制文件路径）

## 修改远程仓库的url

`git remote set-url origin <URL>`

## 把A分支的某个commit放到B分支上

`git checkout <branch-name> && git cherry-pick <commit-id>`

## 展示所有Stash

`git stash list`

## 回到某个Stash

`git stash apply <stash@{n}>`

## 展示某个分支某个文件的内容

`git show <branch-name>:<file-name>`

## 以最后提交的顺序列出所有分支

`git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/`
# Somthing with Git

## `git branch` and `git checkout -b branch_name`
`git checkout -b branch_name` = `git branch branch_name & git checkout branch_name`

## `git merge branch_name`
- FAST-FORWARD 
- NO FAST-FORWARD 

两种方式合并分支的异同：
- 提交对象都会保存；
- 报存提交对象方式不同：快速推进方式是直接在主线（合并主分支）上，添加这些提交对象，即直接移动HEAD指针；而非快速推进方式是将提交对象保存在支线，然后在主线新建一个提交对象，修改HEAD指针及新建提交对象的指针，而且此新建提交对象有两个父提交对象（即有两个parent指针）。
- 合并后分支指向不同：快速推进合并后，两个分支将同时指向最新提交对象，而非快速推进合并后，合并主分支指向新建的提交对象，另一分支指向不变。

## `git branch -d branch_name` delete branch

## `git checkout -b test origin/develop` remote branch
= `git checkout --track origin/develop` use remote branch name as local one 

## `git branch -u origin/develop` set trcked remote branch for current local branch

## `git branch -vv` view remote branch that current one has tracked

## `git push origin --delete test` delete remote branch

## `rebase` 
`git rebase`

has conflict -> fix conflict and then `git rebase --continue`

`git rebase --skip` to skip this patch

`git rebase --abort` to checkout the original branch

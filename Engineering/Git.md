# Git

## Git 概述

Git主要分成3个分区，工作区，暂存区和版本库  

工作区即保存后暂未`git add`的状态  
暂存区是`git add`后未`git commit`的状态  
版本库即`git commit`后的状态  
使用`git push`可将本地的版本库推送到远程版本库  

## Git 常用命令

### 版本合并

- merge

在将一个分支的修改提交到另一分支时，常用`git merge`命令，使用`git merge`会在目标分支创建一个新的commit，这个commit记录了两个分支不同的修改，如果存在冲突，修改先解决冲突，再提交这个commit

- rebase

* 不要通过 rebase 操作已经提交到远程仓库的分支，推荐rebase处理只会影响自己的分支 

由于每次merge都会生成一个commit，所以在多个分支相互影响的情况下，一个分支的commit可能会比较乱，因为其中大部分都是merge commit的信息。  

使用rebase可以将一个分支的某一段提交应用到另一分支上
```git
git rebase [startpointhash] [endpointhash] --onto [branchName]
```

如果某一个分支分支是完全先于当前分支的，可以使用`git rebase [target]`来使当前分支同步到另一分支的修改

使用`rebase -i [startpointhash] [endpointhash]`可以缩减一个分支上的多次commit为一个commit

## 一个推荐的 Git 工作流

- master为线上稳定分支，线上环境应该只部署此分支

- develop分支为产品体验分支，在此分支上可能存在多个feature

- 当开发一个新的功能时，从master分支checkout出一个新的feature分支，开发完成后merge到develop分支上，在产品预览环境测试，没有问题后将此feature分支合并到master

- hotfix分支为快速修复bug分支，hotfix是master上checkout出的分支，修复完成后直接merge到master上

## 推荐工具

- VsCode GitLen
- Git Kraken
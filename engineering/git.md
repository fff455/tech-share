# Git

## Git 概述

Git 主要分成 3 个分区，工作区，暂存区和版本库

工作区即保存后暂未`git add`的状态  
暂存区是`git add`后未`git commit`的状态  
版本库即`git commit`后的状态  
使用`git push`可将本地的版本库推送到远程版本库

## Git 常用命令

### 版本合并

* merge

在将一个分支的修改提交到另一分支时，常用`git merge`命令，使用`git merge`会在目标分支创建一个新的 commit，这个 commit 记录了两个分支不同的修改，如果存在冲突，修改先解决冲突，再提交这个 commit

* rebase
* 不要通过 rebase 操作已经提交到远程仓库的分支，推荐 rebase 处理只会影响自己的分支

由于每次 merge 都会生成一个 commit，所以在多个分支相互影响的情况下，一个分支的 commit 可能会比较乱，因为其中大部分都是 merge commit 的信息。

使用 rebase 可以将一个分支的某一段提交应用到另一分支上

```text
git rebase [startpointhash] [endpointhash] --onto [branchName]
```

如果某一个分支分支是完全先于当前分支的，可以使用`git rebase [target]`来使当前分支同步到另一分支的修改

使用`rebase -i [startpointhash] [endpointhash]`可以缩减一个分支上的多次 commit 为一个 commit

## 一个推荐的 Git 工作流

* master 为线上稳定分支，线上环境应该只部署此分支
* develop 分支为产品体验分支，在此分支上可能存在多个 feature
* 当开发一个新的功能时，从 master 分支 checkout 出一个新的 feature 分支，开发完成后 merge 到 develop 分支上，在产品预览环境测试，没有问题后将此 feature 分支合并到 master
* hotfix 分支为快速修复 bug 分支，hotfix 是 master 上 checkout 出的分支，修复完成后直接 merge 到 master 上

## 推荐工具

* VsCode GitLen
* Git Kraken

## Git 工作流

团队一般为了规范开发，保持良好的代码提交记录以及维护 Git 分支结构清晰，方便后续维护等，都会迫切需要一个比较规范的 Git 工作流。

### Git 分支类型

* master 分支

  master 为产品主分支，该分支为只读唯一分支，也是用于部署生产环境的分支，需确保 master 分支的稳定性。master 分支一般由 release 分支或 hotfix 分支合并，任何情况下都不应该直接修改 master 分支代码。产品的功能全部实现后，最终在 master 分支对外发布，另外所有在 master 分支的推送应该打标签（tag）做记录，方便追溯。master 分支不可删除。

* develop 分支

  develop 为主开发分支，基于 master 分支创建，始终保持最新完成功能的代码以及 bug 修复后的代码。develop 分支为只读唯一分支，只能从其他分支合并，不可以直接在该分支做功能开发或 bug 修复。一般开发新功能时，feature 分支都是基于 develop 分支下创建的。develop 分支包含所有要发布到下一个 release 的代码。feature 功能分支完成后, 开发人员需合并到 develop 分支\(不推送远程\)，需先将 develop 分支合并到 feature，解决完冲突后再合并到 develop 分支。当所有新功能开发完成后，开发人员并自测完成后，此时从 develop 拉取 release 分支，进行提测。release 或 hotfix 分支上线完成后, 开发人员需合并到 develop 分支并推送远程。develop 分支不可删。

* feature 分支

  feature 分支通常为新功能或新特性开发分支，以 develop 分支为基础创建 feature 分支。分支命名: feature/ 开头的为新特性或新功能分支，建议的命名规则: feature/user\_createtime\_feature，例如：feature/ftd\_20201018\_alipay，含义为：开发人员 ftd 在 2020 年 10 月 18 日时创建了一个支付宝支付的功能分支。新特性或新功能开发完成后，开发人员需合到 develop 分支。feature 分支可同时存在多个，用于团队中多个功能同时开发。feature 分支属于临时分支，功能完成后可选删除。

* release 分支

  release 分支为预上线分支，基于本次上线所有的 feature 分支合并到 develop 分支之后，从 develop 分支创建。分支命名: release/ 开头的为预上线分支，建议的命名规则: release/version\_publishtime，例如：release/v2.1.1\_20201018，含义为：版本号 v2.1.1 计划于 2020 年 10 月 18 日时发布。release 分支主要用于提交给测试人员进行功能测试。发布提测阶段，会以 release 分支代码为基准进行提测。测试过程中发现的 bug 在本分支进行修复，上线完成后需合并到 develop/master 分支并推送远程。release 分支属于临时分支，产品上线后可选删除。 当有一组 feature 开发完成后，首先开发人员会各自将最新功能代码合并到 develop 分支。进入提测阶段时，开发组长在 develop 分支上创建 release 分支。 如果在测试过程中发现 bug 需要修复，则直接由开发者在 release 分支修复并提交。当测试完成后，开发组长将 release 分支合并到 master 和 develop 分支，此时 master 为最新可发布代码，用作产品发布上线。

* hotfix 分支

  hotfix 分支为线上 bug 修复分支或叫补丁分支，主要用于对线上的版本进行 bug 修复。分支命名: hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似，建议的命名规则: hotfix/user\_createtime\_hotfix，例如：hotfix/ftd\_20201018\_alipaybugfix，含义为：开发人员 ftd 在 2020 年 10 月 18 日时创建了一个支付宝支付 bug 修复的分支。hotfix 分支用于线上出现紧急问题时，需要及时修复，以 master 分支为基线，创建 hotfix 分支。当问题修复完成后，需要合并到 master 分支和 develop 分支并推送远程。所有 hotfix 分支的修改会进入到下一个 release。hotfix 分支属于临时分支，bug 修复上线后可选删除。

### Git Flow 工作流

我们现在已经了解了 Git 的分支有哪些类型，不过往往在一个团队人数较多，创建的分支也比较多的时候，还是会带来很多分支操作上的困扰。Git Flow 的工作流模式可以从流程上很好的规范开发者。

1. 主分支流程
2. master 分支记录了每次版本发布历史和 tag 标记。
3. develop 分支记录了所有开发的版本历史。
4. develop 分支仅第一次创建时从 master 分支拉取。
5. 开发流程
6. feature 分支是从 develop 分支拉取的分支。
7. 每个 feature 完成后需合并到 develop 分支。
8. 提测发布流程
9. release 分支是在所有功能开发自测完成后，从 develop 分支拉取的分支。
10. release 分支一旦创建后，通常不再从 develop 分支拉取，该分支只做 bug 修复，文档生成和其他面向发布的任务。
11. release 分支测试完成，达到上线标准后，需合并回 master 分支和 develop 分支。
12. bug 修复流程
13. hotfix 分支是在线上出现 bug 之后，从 master 分支拉取的分支。
14. hotfix 分支测试完成后，需合并回 master 分支和 develop 分支。


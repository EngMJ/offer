## git 常用操作

+   参考：[Git常用操作指南](https://www.cnblogs.com/guoyaohua/p/Git-tutorial.html "https://www.cnblogs.com/guoyaohua/p/Git-tutorial.html")

| 命令                      | 说明                          | 示例命令                                  |
|---------------------------|-----------------------------|-------------------------------------------|
| **git init**              | 初始化一个新的 Git 仓库              | `git init`                                |
| **git clone**             | 克隆远程仓库到本地                   | `git clone <repository>`                  |
| **git add**               | 将修改添加到暂存区                   | `git add .` 或 `git add <file>`             |
| **git commit**            | 提交暂存区中的内容到仓库                | `git commit -m "提交信息"`                 |
| **git status**            | 查看工作区和暂存区的状态                | `git status`                              |
| **git log**               | 查看提交历史记录                    | `git log`                                 |
| **git diff**              | 查看文件的改动                     | `git diff`                                |
| **git branch**            | 查看或管理分支                     | `git branch` 或 `git branch <branch>`       |
| **git checkout**          | 切换分支或恢复工作区文件                | `git checkout <branch>`                   |
| **git merge**             | 合并指定分支到当前分支                 | `git merge <branch>`                      |
| **git pull**              | 从远程仓库拉取最新版本并合并到本地分支         | `git pull`                                |
| **git push**              | 将本地提交推送到远程仓库                | `git push`                                |
| **git remote**            | 查看或管理远程仓库                   | `git remote -v`                           |
| **git reset**             | 重置当前 HEAD 到指定状态             | `git reset --hard HEAD`                   |
| **git revert**            | 撤销一次提交（产生一次新的提交）            | `git revert <commit>`                     |
| **git stash**             | 暂时保存当前工作进度（将工作区和暂存区的改动保存起来） | `git stash`                               |
| **git stash list**        | 查看所有已保存的工作进度                | `git stash list`                          |
| **git stash apply**       | 恢复/暂存改动           | `git stash apply [stash@{n}]`              |
| **git cherry-pick**       | 将其他分支的指定提交应用到当前分支           | `git cherry-pick <commit>`                |


**§ git rebase** **作用**：将一个分支的更改合并入另一个分支。

**目的**：

1.  让多个人在同一个分支开发的提交节点形成一条线，而不是多条线
2.  让你提交的commit在该分支的最前面

**使用场景**：

1.  开发分支合并主分支的更新；
2.  不能在主分支上使用rebase，因为会破坏历史记录

**§ merge 和 rebase 的区别** git rebase和git merge做的事其实是一样的。它们都被设计来将一个分支的更改并入另一个分支，只不过方式有些不同。

+   git merge 将两个分支的提交记录按照顺序排序
+   git rebase 把被合并的旁分支的修改直接追加在主分支提交记录后面。

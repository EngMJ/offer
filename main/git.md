## git 常用操作

+   参考：[Git常用操作指南](https://www.cnblogs.com/guoyaohua/p/Git-tutorial.html "https://www.cnblogs.com/guoyaohua/p/Git-tutorial.html")

| 命令             | 作用                               |
|----------------|----------------------------------|
| git status     |                                  |
| git diff       |                                  |
| git add        |                                  |
| git commit     |                                  |
| git push       |                                  |
| git branch     |                                  |
| git checkout   |                                  |
| git pull/fetch | git pull = git fetch + git merge |
| git merge      |                                  |
| git stash      | 本地数据缓存,但不提交                      |

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

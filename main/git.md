## Git 常用操作

+ 参考：[Git常用操作指南](https://www.cnblogs.com/guoyaohua/p/Git-tutorial.html)

### Git 工作流程

```
工作区 (Working Directory)
    ↓ git add
暂存区 (Staging Area / Index)
    ↓ git commit
本地仓库 (Local Repository)
    ↓ git push
远程仓库 (Remote Repository)
```

### 基础命令

| 命令 | 说明 | 示例命令 |
|------|------|----------|
| **git init** | 初始化一个新的 Git 仓库 | `git init` |
| **git clone** | 克隆远程仓库到本地 | `git clone <repository>` |
| **git add** | 将修改添加到暂存区 | `git add .` 或 `git add <file>` |
| **git commit** | 提交暂存区中的内容到仓库 | `git commit -m "提交信息"` |
| **git status** | 查看工作区和暂存区的状态 | `git status` |
| **git log** | 查看提交历史记录 | `git log --oneline --graph` |
| **git diff** | 查看文件的改动 | `git diff` 或 `git diff --staged` |
| **git fetch** | 从远程仓库获取最新版本（不合并） | `git fetch origin` |
| **git pull** | 从远程仓库拉取并合并到本地分支 | `git pull` (等于 fetch + merge) |
| **git push** | 将本地提交推送到远程仓库 | `git push origin <branch>` |
| **git remote** | 查看或管理远程仓库 | `git remote -v` |

### 分支操作

| 命令 | 说明 | 示例命令 |
|------|------|----------|
| **git branch** | 查看/创建/删除分支 | `git branch` / `git branch <name>` / `git branch -d <name>` |
| **git checkout** | 切换分支或恢复工作区文件 | `git checkout <branch>` |
| **git switch** | 切换分支（推荐，更语义化） | `git switch <branch>` / `git switch -c <new-branch>` |
| **git merge** | 合并指定分支到当前分支 | `git merge <branch>` |
| **git rebase** | 变基，将当前分支的提交移到目标分支之后 | `git rebase <branch>` |
| **git cherry-pick** | 将其他分支的指定提交应用到当前分支 | `git cherry-pick <commit>` |

### 撤销与回退

| 命令 | 说明 | 示例命令 |
|------|------|----------|
| **git reset --soft** | 重置 HEAD，保留暂存区和工作区 | `git reset --soft HEAD~1` |
| **git reset --mixed** | 重置 HEAD 和暂存区，保留工作区（默认） | `git reset HEAD~1` |
| **git reset --hard** | 重置 HEAD、暂存区和工作区（危险） | `git reset --hard HEAD~1` |
| **git revert** | 撤销一次提交（产生一次新的提交，安全） | `git revert <commit>` |
| **git restore** | 恢复工作区文件（推荐） | `git restore <file>` / `git restore --staged <file>` |

### 暂存操作

| 命令 | 说明 | 示例命令 |
|------|------|----------|
| **git stash** | 暂存当前工作进度 | `git stash` 或 `git stash -m "message"` |
| **git stash list** | 查看所有暂存的工作进度 | `git stash list` |
| **git stash apply** | 恢复暂存的改动（不删除 stash） | `git stash apply [stash@{n}]` |
| **git stash pop** | 恢复暂存的改动（删除 stash） | `git stash pop` |
| **git stash drop** | 删除指定的 stash | `git stash drop stash@{n}` |

### 标签管理

| 命令 | 说明 | 示例命令 |
|------|------|----------|
| **git tag** | 查看所有标签 | `git tag` |
| **git tag \<name\>** | 创建轻量标签 | `git tag v1.0.0` |
| **git tag -a** | 创建附注标签 | `git tag -a v1.0.0 -m "版本说明"` |
| **git push --tags** | 推送所有标签到远程 | `git push origin --tags` |

---

## merge 和 rebase 的区别

### 一句话理解

- **merge**：把两条路合成一条，保留"岔路口"
- **rebase**：把你走过的路"搬"到别人的路后面，假装从没分过岔

### 生活化比喻

想象你和同事小王同时从 `v1.0` 版本开始各自开发：

```
         你的工作: A → B
        /
v1.0 ──
        \
         小王的工作: C → D（已合入主分支）
```

现在主分支已经有了小王的代码，你需要把自己的工作合进去：

**用 merge（合并）**：就像两条河流汇合

```
         你的工作: A → B ─┐
        /                  ↘
v1.0 ──                     合并点 → 继续开发
        \                  ↗
         小王的工作: C → D ─┘
```

历史记录会显示：这里曾经分叉过，然后又合在一起了。

**用 rebase（变基）**：就像把你的工作"剪下来"，"粘"到小王后面

```
v1.0 → C → D → A' → B'
       ↑小王   ↑你（重新应用）
```

历史记录会显示：好像大家一直在一条线上顺序开发，从没分过岔。

### 对比总结

| | git merge | git rebase |
|--|-----------|------------|
| **像什么** | 河流汇合 | 剪切粘贴 |
| **历史记录** | 真实，保留分叉 | 整洁，一条直线 |
| **操作** | 产生一个新的"合并提交" | 把你的提交"复制"到目标分支后面 |
| **原提交** | 不变 | 会产生新的 commit hash |
| **安全性** | 安全 | 别在公共分支上用！ |

### 什么时候用什么？

```
场景                              推荐
─────────────────────────────────────────
把 feature 分支合入 main         → merge
同步 main 的更新到自己的分支      → rebase
整理自己的提交历史                → rebase -i
多人协作的公共分支                → 只用 merge
```

### 黄金法则

> **不要在公共分支上 rebase！**
> 
> 因为 rebase 会改变 commit 的 hash，如果别人已经基于原来的提交开发，你 rebase 后他们的代码就会乱套。

---

## git rebase 详解

**交互式 rebase**（整理提交历史）：

```bash
git rebase -i HEAD~3  # 整理最近 3 次提交
```

可用操作：
- `pick`: 保留该提交
- `reword`: 修改提交信息
- `squash`: 合并到前一个提交（多个小提交合成一个）
- `drop`: 删除该提交

---

## 冲突解决

当 merge 或 rebase 产生冲突时：

```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑冲突文件，解决冲突标记
<<<<<<< HEAD
当前分支的内容
=======
合并分支的内容
>>>>>>> branch-name

# 3. 标记冲突已解决
git add <conflicted-file>

# 4. 继续操作
git merge --continue   # 或
git rebase --continue

# 放弃操作
git merge --abort      # 或
git rebase --abort
```

---

## 常用配置

```bash
# 设置用户信息
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 设置默认分支名
git config --global init.defaultBranch main

# 设置别名
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
```

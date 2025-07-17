[TOC]

### Overview

几乎每一种版本控制系统都以某种形式支持分支，一个分支代表一条独立的开发线。

Git 分支实际上是指向更改快照的指针。

### Create branch

创建新分支并切换到该分支：

```
git checkout -b <branchname>
```

### Check branch

查看所有分支：

```
git branch
```

查看远程分支：

```
git branch -r
```

查看所有本地和远程分支：

```
git branch -a
```

### Merge branch

将其他分支合并**到当前分支**：

```
git merge <branchname>
```

例如，切换到 main 分支并合并 feature-xyz 分支：

```
git checkout main
git merge feature-xyz
```

### Solve conflict

当合并过程中出现冲突时，Git 会标记冲突文件，你需要手动解决冲突。

打开冲突文件，按照标记解决冲突。

标记冲突解决完成：

```
git add <conflict-file>
```

提交合并结果：

```
git commit
```

### Delete branch

删除本地分支：

```
git branch -d <branchname>
```

强制删除未合并的分支：

```
git branch -D <branchname>
```

删除远程分支：

```
git push origin --delete <branchname>
```


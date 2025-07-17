[TOC]

# Overview

![img](https://www.runoob.com/wp-content/uploads/2015/02/git-command.jpg)

```shell
$ git init    
$ git add .    
$ git commit 
```

## Working Directory

工作目录是你在本地计算机上看到的项目文件。它是你实际操作文件的地方，包括查看、编辑、删除和创建文件。所有对文件的更改首先发生在工作目录中。

在工作目录中的文件可能有以下几种状态：

- **未跟踪（Untracked）**：新创建的文件，未被 Git 记录。
- **已修改（Modified）**：已被 Git 跟踪的文件发生了更改，但这些更改还没有被提交到 Git 记录中。

## Staging Area

暂存区，也称为索引（Index），是一个临时存储区域，用于保存即将提交到本地仓库的更改。你可以选择性地将工作目录中的更改添加到暂存区中，这样你可以一次提交多个文件的更改，而不必提交所有文件的更改。

- 使用 `git add <filename>` 命令将文件从工作目录添加到暂存区。
- 使用 `git add .` 命令将当前目录下的所有更改添加到暂存区。

## Local Repository

本地仓库是一个隐藏在 `.git` 目录中的数据库，用于存储项目的所有提交历史记录。每次你提交更改时，Git 会将暂存区中的内容保存到本地仓库中。

使用 `git commit -m "commit message"` 命令将暂存区中的更改提交到本地仓库。

## 文件状态的转换流程

### Untracked

 新创建的文件最初是未跟踪的。它们存在于工作目录中，但没有被 Git 跟踪。

```
touch newfile.txt  # 创建一个新文件
git status         # 查看状态，显示 newfile.txt 未跟踪
```

### Tracked

通过 `git add` 命令将未跟踪的文件添加到暂存区后，文件变为已跟踪状态。

```
git add newfile.txt  # 添加文件到暂存区
git status           # 查看状态，显示 newfile.txt 在暂存区
```

### Modified

对已跟踪的文件进行更改后，这些更改会显示为已修改状态，但这些更改还未添加到暂存区。

```
echo "Hello, World!" > newfile.txt  # 修改文件
git status                          # 查看状态，显示 newfile.txt 已修改
```

### Staged

使用 `git add` 命令将修改过的文件添加到暂存区后，文件进入已暂存状态，等待提交。

```
git add newfile.txt  # 添加文件到暂存区
git status           # 查看状态，显示 newfile.txt 已暂存
```

### Committed

使用 `git commit` 命令将暂存区的更改提交到本地仓库后，这些更改被记录下来，文件状态返回为已跟踪状态。

```
git commit -m "Added newfile.txt"  # 提交更改
git status                         # 查看状态，工作目录干净
```

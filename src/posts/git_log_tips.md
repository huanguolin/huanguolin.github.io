---
title: git log tips
description: "列举我常用的 git log 命令，备忘"
date: 2024-03-03T12:15:00
thumb: "git-log-tips.png"
tags:
    - git
    - git-log
    - tips
---

列举我常用的 git log 命令，备忘。

## 1. 单行显示

```sh
git log --oneline

# 它很常用，所以我一般把它配成了一个别名
git config --global alias.ls log --oneline
# 然后这样用
git ls

# git log --oneline 它等价于
git log --pretty=oneline --abbrev-commit

# 使用 --pretty 可以自定义格式，格式选项参考文档：https://git-scm.com/docs/git-log#_pretty_formats
git log --pretty='%h | %d %s (%cr) [%an]'
```

## 2. 控制显示个数

```sh
# 显示最近的三次提交
git log -n 3

# 可以简写为
git log -3
```

## 3. 显示变动文件

```sh
git log --name-only

# 它会在每个 commit log 下列出涉及的文件，如：
.gitignore
src/_includes/partials/header.njk
src/assets/img/favicon.png
src/assets/img/icon.svg
```

但是往往我们还希望知道，文件状态是添加、修改、还是删除？那就用下面这个：

```sh
git log --name-status

# 它会在每个文件前标明状态(M: 修改，D: 删除，A: 添加)，如：
M       .gitignore
M       src/_includes/partials/header.njk
D       src/assets/img/favicon.png
A       src/assets/img/icon.svg
```

## 4. 限定文件、文件夹

```sh
# 很简单，后面带上文件或者文件夹路径即可
git log src/assets/img/icon.svg
git log src/assets/

# 可以列多个路径
git log src/index.js src/utils/helper.js

# 当然也支持通配符
git log src/**/*.png

# 还可以反过来，排除某些路径。比如看 src 下但不含 src/assets/ 的历史
git log src/ ':(exclude)src/assets'

# 还可配合其他参数，比如看文件变更状态(ls 是上面介绍的一个别名)：
git ls --name-status src/assets/img/icon.svg
```

还有一个常见的需求就是找到，一个文件是什么时候加进去的？或者什么时候被删除的？
使用 `--name-status` 也可以，就是效率不高（文件变动的历史多的话，就不容易找了）。更高效的方式是使用下面要介绍参数：

## 5. 按文件变动状态过滤

```sh
git log --diff-filter=[(A|D|M|R)]
# A,M,D 三个上面已经说过了，R：是重命名, 还有更多状态标志请看文档：https://git-scm.com/docs/git-log#Documentation/git-log.txt---diff-filterACDMRTUXB82308203

# 我们想找到 src/index.js 是什么时候加进去的:
git log --diff-filter=A src/index.js

# 被删除或者重命名：
git log --diff-filter=DR src/index.js
```

## 6. 过滤 commit message

```sh
git log --grep feat

# 用这个可以找到特定 commit message 对应的 commit, 比如找 revert 记录：
git log --grep revert
```

## 7. 过滤 merge commit 或者 non-merge commit

有时要查合并历史，或者排除合并历史，显示变动“干货”，那么这个很有用：
```sh
git log --merges # 等同于 --min-parents=2
git log --no-merges
```

----
更多内容可以参考官方文档： [git-log](https://git-scm.com/docs/git-log)


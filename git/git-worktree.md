## git worktree的用法

`git worktree` 相当于对同一个 `.git` 仓库创建多个目录，这样便可以在目录之前随意切换，这样在一些main分支需要紧急hotfix时，不需要使用 `git stash` + `git stach pop` 进行暂存再切换到main分支的操作了。

演示如下：

```bash
# 创建一个演示目录 就称之为 `my-proj` 项目吧
mkdir my-proj
# 初始化
git init
# 随意创建一个项目
echo 'this is a file' >> a.txt
git commit -am 'add a.txt'

# 显示所有worktree
# 此时只有一个默认的worktree
git worktree list
# 打印如下，只有一个worktree 目录相当于 my-proj 对应main分支
# /Users/xyz/Documents/2023/my-proj 42abc39 [main]

# 创建一个新的worktree 这同时也会创建一个新的分支 feat1
git worktree add ../feat1

# 显示所有worktree
# 此时便有2个worktree了
git worktree list
# /Users/xyz/Documents/2023/my-proj  42abc39 [main]
# /Users/xyz/Documents/2023/feat1    fd8ae8e [feat1]

# 并且也存在2个分支
git branch -a
# main
# feat1

# 使用 `cd xx` 切换到 feat1 worktree
cd ../feat1
# 在feat1目录中写入一个新的文件
echo 'this is b file' >> b.txt
# 😎但是我们此时不提交该文件
# 直接就可以切换到 my-proj worktree了
cd ../my-proj

# 在 `my-proj` worktree基础上再创建一个新的worktree
# 会对应创建 `feat2` 分支🍓，是下面写法的简写
# git worktree add -b feat2 ../feat2
git worktree add ../feat2
# 同时这也会创建一个新的分支 'feat2'
# 显示所有worktree
git worktree list
# /Users/xyz/Documents/2023/my-proj   fd8ae8e [main]
# /Users/xyz/Documents/2023/feat1     42abc39 [feat1]
# /Users/xyz/Documents/2023/feat2     fd8ae8e [feat2]
# 我们切换到 feat2 然后写入一个新的文件
cd ../feat2
echo 'this is c file' >> c.txt
git commit -am 'feat2: add c.txt in worktree feat2'

# 将feat2 worktree功能合并到 main worktree中
git rebase main

# 切换到 main worktree
cd ../my-proj
git merge feat2

# 然后切换到 feat1 worktree
# 将之前没有提交的文件提交
cd ../feat1
git commit -am 'feat1: add b.txt in worktree feat1'
git rebase main

# 最后切换到 main worktree
cd ../my-proj
git merge feat1

# 最终得到的文件记录为
git log

# 'add a.txt'
# 'feat2: add c.txt in worktree feat2'
# 'feat1: add b.txt in worktree feat1'

# 为了节省磁盘空间，可以将不使用的worktree删除
git worktree remove feat1
git worktree remove feat2
git worktree prune

# 至此 就只剩下main worktree了
git worktree list
# /Users/xyz/Documents/2023/my-proj     42abc39 [main]
```

当然你也可以给已经存在的分支添加worktree:（不过用起来可能有点怪😅）

```bash
mkdir my-proj
git init
echo 'this is a file' >> a.txt
git commit -am 'add a.txt'
git worktree list
# /Users/xyz/Documents/2023/my-proj  f2648b1 [main]

# 创建一个新的分支
git checkout -b new-feature
echo 'this is b file in new feat' >> b.txt

# 此时你想回到main分支修改点东西
# 你有不想使用 git stash + git stash pop 操作
# 可以基于main分支创建一个新的worktree
git worktree add ../hotfix main
git worktree list
# 🚨 此时 my-proj目录对应的是 new-feature分支
# /Users/xyz/Documents/2023/my-proj  f2648b1 [new-feature]
# /Users/xyz/Documents/2023/hotfix   f2648b1 [main]
cd ../hotfix # 对应main分支
# 做其余的操作
```

参考文章：

1. [如何不切换Git分支，同时在多个分支上工作？](https://blog.didispace.com/git-work-on-different-branch/)
2. [Git Worktree的高级使用介绍](https://blog.didispace.com/git-worktree-2/)
3. [Git Worktree的使用](https://jasonkayzk.github.io/2020/05/03/Git-Worktree%E7%9A%84%E4%BD%BF%E7%94%A8/)
4. [Practical Guide to Git Worktree - @dev.to](https://dev.to/yankee/practical-guide-to-git-worktree-58o0)
5. [Cameron McKenzie - @youtube](


2023年05月21日02:03:43
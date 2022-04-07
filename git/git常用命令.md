```
# 删除本地分支 -D 表示不管本地和远程分支是否同步 直接删除
git branch -d [branch_name]
git branch -D [branch_name]
# 删除远程分支并且push到远程 进行同步
git push origin --delete [branch_name]

# 删除远程分支
//git branch -d -r origin/jianguo_prod_102_beiyong

# zsh git快捷键
# 查看本地分支
gb   (git branch)
# 查看远程分支
gbr (git branch -r)
# 查看所有分支
gba (git branch -a)


# 撤销当前更改
git checkout .

# 暂存更改
git stash
# 将暂存弹出
git stash pop

# https://blog.csdn.net/wangjia55/article/details/8793577
# 给当前分支打标签
git tag -a v0.1.2 -m “0.1.2版本”
# 在控制台打印当前仓库的所有标签
git tag #
# 将 v0.1.2标签提交到git服务器
git push origin v0.1.2
# 删除本地tag
git tag -d v0.1.2
# 删除远程tag
git push origin :refs/tags/v0.1.2

# 将本地分支推送到远程分支
git push origin [local_branch_name]:[remote_branch_name]

# 强制覆盖使用local_branch_name覆盖线上remote_branch_name
git push origin [local_branch_name]:[remote_branch_name] --force


# 分支重命名 rename
# 对当前分支重命名
git branch -m [new_branch_name]
# 对其它分支重命名
git branch -m [other_branch_old_name] [other_branch_new_name]
```



```
# 合并其他分支的某个特定commit 到当前分支
# 比如当前分支为 dev_1
# 希望合并分支 dev_2 中的commit c5f243e3
# 可以使用cherry-pick 命令
git cherry-pick c5f243e3

// 如果有冲突 解决冲突
// 然后
git add .
git commit
```



拉取远程某个分支

```
git fetch origin [romote_branch_name]
```

**拉取分支并且切换分支**

```
git checkout -b [romote_branch_name] origin/[romote_branch_name]
```



将所有的分支推送到remote

```bash
git remote add origin git@192.168.2.114:[username]/[project_name].git
git push --all origin
```



## 拉取远程分支

```
# 第1种 推荐
git fetch origin dev
git rebase origin/dev

# 第2种 会额外产生交叉线
git pull origin dev

# 第3种
git pull --rebase origin dev

// 如果碰到 First, rewinding head to replay your work on top of it...
// 使用下面2个命令
// https://blog.csdn.net/sanbingyutuoniao123/article/details/78187229
git fetch origin
git reset --hard origin/<branch>
```



## 功能分支rebase

```
// 假设有2个分支：master & feature_branch

// 先将主分支从远程同步下来 防止别人提交和当前有冲突
git fetch origin dev
git rebase origin/dev

// 先切到功能分支
git checkout feature_branch

git rebase master

git checkout master

// 合并功能分支
git merge feature_branch
```



## 如何合并另一个分支的多个提交

对于单个提交，可以使用 **`cherry-pick`** 的方式，对于多个连续的提交，使用 **`rebase`** 则更好

- [git rebase 的用法 - 简书](https://www.jianshu.com/p/4a8f4af4e803)

语法：

```
# startCommit是开区间 即不包含这个提交点 可以使用 824335b^ 添加 '^' 表示上一个提交的意思
# endCommit是闭区间 即包含这个点
git  rebase  (startCommit)  [endCommit]   --onto branch_you_want_to_rebase
```

比如有 A, B 2个互不关联的分支，A的提交记录为(使用 **`git log`** 查看)：

```
H: commit ee988180bf17758de1ee53cccde0582107511fea
    build: iOS 将1.0.2版本修改为1.0.3版本

G: commit 07669521a0b5489906e2b51aebb9ba75220baf87
    iOS ClientDataVM 删除PotatsoKit 的引用 避免打包报错

F: commit e13b245e80b3bb763707404773f5c62cedd874a4
    style: SiteNavigation2 修改item样式 适配ipad 上下区域点击返回效果

E: commit ac23b61cbbc8eadb6e9096d84d44d376130070a5
    build: iOS pod install 新的podfile

D: commit 4a6e16cbcfee313bdd2468e1716e9cb6d0a1ed3e
    fix: iOS修改配置文件加载逻辑

C: commit 72ebcb1e6404109bbb1698147cea5bf8d3a45ca3
    build: 更新Pods.xcodeproj 文件

B: commit 824335bd8100ae192acd120a7b6db7959a4d149e
    fix: iOS修复.cn在智能代理下不能翻墙的问题

A: commit e626f623926984dd548e3421cb9e18d1faf41da6
    fix: iOS开屏广告业务逻辑修改增加视频播放
```

比如现在想要合并A分支中的 **`B-G`** 这段提交到B分支，可以使用下面命令：

```
# 切换到A分支上
git checkout A
# rebase 将B-G提交点 合并到B分支上
git rebase 824335b^ 0766952 --onto B

# 合并过程中 可能出现文件冲突
# 解决完冲突后 将文件暂存
git add .
# 然后继续
git rebase --continue

# 有可能还是会出现冲突
# 则继续向上面一样解决冲突 然后继续
git add .
git rebase --continue

# 至到所有冲突都解决完成
# 这个时候我们只是把A分支的B-G提交提取出来了 并没有合并到B分支上
# 要合并到B分支则 还需要切换到B分支
# 切换的时候会提示 git branch <new-branch-name> 0384821
# 上面的 0384821 表示git hash, 我们需要把这个提交添加到B中
git checkout B
git reset --hard 0384821   # 大功完成
```





## 如何取消追踪已经提交的文件

先将不想追踪的文件添加到 **`.gitignore`** 中，比如

```
Potatos2_Swift/SvpnClient/PacketTunnel/Vendor/build
Potatos2_Swift/SvpnClient/SvpnClient.xcworkspace/xcuserdata
Potatos2_Swift/SvpnClient/SvpnClient/Lib/ShadowPath/ShadowPath.xcodeproj/xcuserdata
Potatos2_Swift/SvpnClient/SvpnClient/Lib/ICKit/ICKit.xcodeproj/xcuserdata
Potatos2_Swift/SvpnClient/SvpnClient.xcodeproj/xcuserdata
Potatos2_Swift/SvpnClient/Pods/Pods.xcodeproj/xcuserdata
```

然后将这些文件从git中移除：

```
git rm -r --cached Potatos2_Swift/SvpnClient/SvpnClient.xcworkspace/xcuserdata
git rm -r --cached Potatos2_Swift/SvpnClient/SvpnClient/Lib/ShadowPath/ShadowPath.xcodeproj/xcuserdata
git rm -r --cached Potatos2_Swift/SvpnClient/SvpnClient/Lib/ICKit/ICKit.xcodeproj/xcuserdata
git rm -r --cached Potatos2_Swift/SvpnClient/SvpnClient.xcodeproj/xcuserdata
git rm -r --cached Potatos2_Swift/SvpnClient/PacketTunnel/Vendor/build
git rm -r --cached Potatos2_Swift/SvpnClient/Pods/Pods.xcodeproj/xcuserdata
```



## 对比拉取分支差异

比如A开发人员将dev分支的功能提交到了远程分支，这时对远程分支拉取，拉取后查看本地分支和远程分支的差异

```
# 拉取远程分支 变化
# 注意这里没有使用 git pull
# git pull 会直接拉取 然后进行合并操作
git fetch origin dev

# 查看本地分支和远程分支之间的差异
git log -p dev origin/dev

# 在合并前先将commit hash 记下来 这样下面合并的时候 出错了就可以reset
git log

# 然后进行合并操作
git merge origin/dev
```



另外还可以使用下面方式：

```
# 实践发现并不好用
git pull --rebase
```

 

## 撤销上一次commit

注意：

- **`--soft`** 和 **`--hard`** 区别： soft 表示保留更改，hard表示直接去除上一次的任何更改

```
git reset --soft HEAD~
```



## 如何将本地仓库提交到多个不同的远程仓库上？

可以使用这个命令添加远程仓库：

```
git remote add [remove_git_name] https://xxx.git
```

比如有2个远程地址：

```
// 第一个 远程仓库名叫 origin
git remote add origin https://www.github.com/projectName.git

// 第一个 远程仓库名叫 origin2
git remote add origin2 https://www.mayun.com/projectName.git

// 将分支全部上传
git push gogs --all
```

可参考：

- [git从本地同时提交到多个远程仓库 - csdn](https://blog.csdn.net/StrutsTwo/article/details/96901843)

- [教你一招解决Git时提交到多个远程仓库 - csdn](https://blog.csdn.net/bigbear00007/article/details/103570928)



## 删除远程分支

```
git push origin --delete [remote_branch_name]
```



##  查看所有的git操作

包含删除分支的记录

```
// 最近前n步git记录
git reflog {n}
```

比如：

```
git reflog 6

// 打印
b967f89 HEAD@{2}: reset: moving to HEAD
b967f89 HEAD@{3}: commit: refactor: home 加速失败提示框提示内容进行修改
f84b736 HEAD@{4}: reset: moving to origin/dev
0c489f9 HEAD@{5}: rebase finished: returning to refs/heads/dev
0c489f9 HEAD@{6}: rebase: refactor: home 加速失败是提示文字内容进行修改
2531af4 HEAD@{7}: rebase: refactor: 添加 staticUrl 用于获取静态资源的地址
```

现在要回到 **`HEAD@{7}`** 处

```
git checkout HEAD@{7}

git switch -

# 创建新的分支
git branch dev-url 0c489f9
```

参考：

1. [git reflog用法](https://www.softwhy.com/article-8573-1.html)



## 查看远程分支提交记录

```
git log remotes/origin/[branch_name]
```

参考：

- [git 查看远程仓库log - csdn](https://blog.csdn.net/u012830148/article/details/82115770)



## 重命名本地分支名

```
git branch -m old_branch_name new_branch_name

// 如果要修改当前分支的名字 可以直接
git branch -m new_branch_name
```

- [How to rename a local Gir branch](https://stackoverflow.com/a/6591218)



## 修改上一次提交的message内容

有时候在 **`git commit -m 'xxx'`** 提交的消息写得不是很好，想要修改该提交信息，可以使用

```bash
git commit --amend
```



## git zsh 别名快捷键



- **`gcmsg`**: **`git commit -m`**, eg: **`gcmsg 'add new file'`**
- **`gaa`**: **`git add .`**
- **`gco`**: **`git checkout`** eg: **`gco master`** 切回master分支，这个还有一个快捷键 **`gcm`**



可以参考：

- [zsh自带的git aliases](https://blog.niclin.tw/2017/07/27/oh-my-zsh-%E8%87%AA%E5%B8%B6%E7%9A%84-git-aliases/)





## 🚀 git worktree

在本地多分支开发时，有可能当前分支有东西在修改，然后需要切到另外一个分支去操作，一般我们会使用 `git stash` 命令，比如：

**假设当前分支在 dev, dev有文件没有提交，此时我们需要切换到master分支**

```bash
# dev 分支 保存到暂存区
git stash

# 切换到master
git checkout master

# master操作完成后，再切换到dev
git checkout dev
# 从暂存区中取出文件继续工作
git stash pop
```

使用 `git worktree` 则会方便很多：假设我们的项目名称叫做 `worktree-demo`

```bash
# 在dev分支
# 添加一个master worktree
git worktree add ../master-branch master

# 查看当前worktree列表
git worktree list
# 打印
/Users/xxx/Documents/2022/worktree-demo  2b081ec [dev]
/Users/xxx/Documents/2022/master-tree    2b081ec [master]

# 切换到master分支 直接cd即可切换到master分支
cd ../master-tree
# 在master上进行操作
echo 'file2 master' > file2.txt
git add .
git commit -m 'add file2 to master'

# 修改完成后切换到 dev 分支 也是使用cd即可
cd ../worktree-demo

# 如果不需要worktree 可以将其移除
git worktree remove master-tree
```

如果想要了解更多worktree命令，可以使用 `git worktree --help`

```bash
NAME
       git-worktree - Manage multiple working trees

SYNOPSIS
       git worktree add [-f] [--detach] [--checkout] [--lock] [-b <new-branch>] <path> [<commit-ish>]
       git worktree list [--porcelain]
       git worktree lock [--reason <string>] <worktree>
       git worktree move <worktree> <new-path>
       git worktree prune [-n] [-v] [--expire <expire>]
       git worktree remove [-f] <worktree>
       git worktree repair [<path>...]
       git worktree unlock <worktree>
```




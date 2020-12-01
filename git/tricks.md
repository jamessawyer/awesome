## 将默认master修改为main

因为github将主分支名 **`master`** 修改为 **`main`**, 如果本地每次创建git仓库，都默认创建的是master分支，如果修改，则需要使用

```shell
# 将当前分支(master) 修改为 main
git branch -M main
```

这样很不方便，可以使用下面方式，设置默认分支名：

```shell
git config --global init.defaultBranch main
```

**注意：上面方法需要git `2.28` 版本以上**，[mac 升级git](https://juejin.im/post/6844903762029445134)

参考来源：

- [[Easily rename your Git default branch from master to main](https://www.hanselman.com/blog/easily-rename-your-git-default-branch-from-master-to-main)](https://www.hanselman.com/blog/easily-rename-your-git-default-branch-from-master-to-main)


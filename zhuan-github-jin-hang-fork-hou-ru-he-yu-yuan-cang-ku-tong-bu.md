# 【转】Github进行fork后如何与原仓库同步



> 在 Gitbub 上, fork 别人的代码提交pr, 或者修改成自己的需要的内容是很常见的. 如何在 fork 别人代码之后, 重新拉取别人分支上的代码是一个问题.

### [\#](https://nfangxu.com/git/how_to_synchronize_with_the_original_warehouse_after_github_fork.html#%E6%96%B9%E5%BC%8F-1-%E6%9A%B4%E5%8A%9B%E5%88%A0%E9%99%A4-%E9%87%8D%E6%96%B0fork)方式 1: 暴力删除, 重新fork <a id="&#x65B9;&#x5F0F;-1-&#x66B4;&#x529B;&#x5220;&#x9664;-&#x91CD;&#x65B0;fork"></a>

> 如果你只是提交了一个小bug, 或者你提交的代码, 已经提交到了别人代码的主仓库, 你可以使用这个方法.

1. 在你fork仓库的setting的最下面, delete这个仓库
2. 进入别人的仓库, 重新fork

### [\#](https://nfangxu.com/git/how_to_synchronize_with_the_original_warehouse_after_github_fork.html#%E6%96%B9%E5%BC%8F-2-%E9%80%9A%E8%BF%87-upstream-%E5%90%8C%E6%AD%A5)方式 2: 通过 upstream 同步 <a id="&#x65B9;&#x5F0F;-2-&#x901A;&#x8FC7;-upstream-&#x540C;&#x6B65;"></a>

> 通过设置 upstream \(中文名: 上游分支\), 同步远程分支

#### [\#](https://nfangxu.com/git/how_to_synchronize_with_the_original_warehouse_after_github_fork.html#%E8%AE%BE%E7%BD%AE-upstream)设置 upstream <a id="&#x8BBE;&#x7F6E;-upstream"></a>

* 查看是否设置 upstream 分支

```text
git remote -v
```

如果没有设置, 你会看到以下信息

```text
origin	git@github.com:nfangxu/hyperf.git (fetch)
origin	git@github.com:nfangxu/hyperf.git (push)
```

* 设置 upstream

> 一般情况下，设置好一次 upstream 后就无需重复设置。

```text
git remote add upstream https://github.com/hyperf/hyperf.git
```

* 查看是否设置成功

```text
git remote -v
```

如果成功设置, 你会看到以下信息

```text
origin	git@github.com:nfangxu/hyperf.git (fetch)
origin	git@github.com:nfangxu/hyperf.git (push)
upstream	https://github.com/hyperf/hyperf.git (fetch)
upstream	https://github.com/hyperf/hyperf.git (push)
```

#### [\#](https://nfangxu.com/git/how_to_synchronize_with_the_original_warehouse_after_github_fork.html#%E5%90%88%E5%B9%B6%E4%BB%A3%E7%A0%81)合并代码 <a id="&#x5408;&#x5E76;&#x4EE3;&#x7801;"></a>

* 获取原仓库的更新

```text
git fetch upstream
```

* 切换到指定分支\(一般是 `master`\)

```text
git checkout master
```

* 合并远程代码

```text
git merge upstream/master
```

* 将代码同步到自己的仓库

```text
git push origin master
```

来源：[https://nfangxu.com/git/how\_to\_synchronize\_with\_the\_original\_warehouse\_after\_github\_fork.html](https://nfangxu.com/git/how_to_synchronize_with_the_original_warehouse_after_github_fork.html)


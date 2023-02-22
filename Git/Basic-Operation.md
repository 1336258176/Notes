# 基本命令

---

## Contents

- [基本命令](#基本命令)
- [远程仓库](#远程仓库)
- [分支管理](#分支管理)
- [标签管理](#标签管理)
- [参考资料](#参考资料)

## 基础命令

- 命令
  > `git init` `git add <file>` `git commit -m <message>`
  >
  > `git status`[^1] `git diff`[^2] `git log`[^3]
  >
  > `git reset --hard <commit id>`[^4] `git reset HEAD <file>`[^5]
  >
  > `git reflog`[^6] `git checkout -- <file>`[^7]
  >
  > `git rm <file>` `git mv`[^8]

## 远程仓库

- **_本地 Git 仓库和 GitHub 之间通过 SSH 加密_**

- 命令
  > `ssh-keygen -t rsa -C "youremail@example.com"`[^9]
  >
  > `git remote add <remote_repo_name> <remote_URL>` `git remote`[^10]
  >
  > `git push <远程主机名> <本地分支名>:<远程分支名>`
  >
  > `git pull <远程主机名> <远程分支名>:<本地分支名>`[^11]

## 分支管理

- **_Git 把每次修改串成一条时间线，这根线就是 `Master` 主支，`HEAD` 指针指向当前分支_**
- **_实际开发中，master 分支应是非常稳定的，所以应该在开发时新建一个分支，让开发成员在该分支上分支工作，最后合并成新版本_**
- **_开发新功能时，最好新建一个分支，以防因突然情况导致不得不放弃该功能_**

- 命令
  > `git branch`[^12] `git checkout`[^13] `git merge <branch>`[^14] `git switch`[^15] `git stash`[^16]
  >
  > `git checkout -b <local_branch> <remote>/<remote_branch>`[^17]
  >
  > `git branch --set-upstream <local_branch> <remote>/<remote_branch>`[^18]

## 标签管理

- **_其实就是”快照“，与 commit 挂钩_**

- 命令
  > `git tag`[^19] `git push <remote> :refs/tags/<tag>`[^20]
  >
  > `git push <remote> <tag>`[^21] `git push <remote> --tags`[^22]

---

## 参考资料：

- [廖雪峰的 git 教程](https://www.liaoxuefeng.com/wiki/896043488029600)

[^1]: 查看仓库当前状态
[^2]: 比较文件的不同，即暂存区和工作区的差异
[^3]: 查看提交日志(`--prettty=online`),`--graph`查看分支合并图
[^4]: 退回版本（~~其实还可以回去~~），HEAD（实际是指针）指向的就是当前版本
[^5]: 把**暂存区**的修改退回到**工作区**，HEAD 表示最新版本
[^6]: 查看命令历史（用于查看`commit id`，返回版本）
[^7]: 撤销**工作区**修改（让文件回到最近一次`git add`或`git commit`时的状态）
[^8]: 移动或重命名工作区文件
[^9]: 创建 SSH 密钥，用户目录中.ssh 里面有 id_rsa（私钥）和 id_rsa.pub（公钥）
[^10]: `-v`显示所有远程仓库 `show <remote>`显示某个远程仓库信息 `rm <remote>`删除 `rename <old_name> <new_name>`重命名
[^11]: 用于从远程获取代码并合并本地的版本
[^12]: 查看或创建分支
[^13]: 切换分支,`-b <branch>`创建并切换到该分支,`-d <branch>`删除分支
[^14]: 合并分支到当前分支
[^15]: 切换分支,`-c`创建并切换到新分支
[^16]: 相当于“快照”,`list`查看所有“快照”,`apply`返回“快照”但不删除快照,`drop`删除”快照“,`pop`返回并删除“快照”
[^17]: 在本地创建和远程分支对应的分支（**二者名称最好相同**）
[^18]: 建立本地分支和远程分支的关联（可直接`git pull`）
[^19]: 查看或为本次分支最近的一次 commit 打标签,`<tag> <commit_id>`为某次特定的 commit 打标签,`-d <tag>`删除本地标签
[^20]: 删除远程标签
[^21]: 推送某个标签
[^22]: 推送全部标签

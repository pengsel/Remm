# Git

<<<<<<< HEAD
### .gitignore

忽略一个文件夹：文件夹名

忽略一个文件：*.文件后缀

如果想忽略一个已经跟踪的文件：

```
git update-index --assume-unchanged FILENAME
```
=======
### 撤回修改

1. 如果还未放入暂存区

```git
git checkout -- file
```

2. 如果放入暂存区，还未commit

```
git reset HEAD file
git checkout -- file
```

3. 已经commit，回退上一个版本

```
git reset --hard HEAD^
```

- `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。



git remote ：查看远程库的信息

git remote -v：查看更详细的远程库的信息

origin 相当于自己的远程仓库

upstream 相当于公共的远程仓库

### 工作流程

整体上是一个环路：

origin 发出merge request到upstream；

本地应该有一个baseline分支与upstream分支对齐；

本地的local_dev在提交了一个commit之后，应该rebase baseline，使得自己的commit在整个项目的commit之前。

##### 具体步骤如下：

1. 建立和远程分支的链接：

```
git branch --set-upstream-to=origin/<branch> dev
```

2. 列出所有分支：

```
git branch -av
```

3. 创建一个新的basline分支与us/master对齐

```
git checkout -b baseline --track us/master
```

4. 拉取远程分支到本地

```
git pull us master
```

5. 新建并切换到本地分支<local_dev>，也是从us/master

```
git checkout -b local_dev --track us/master
```

6. 在本地<local_dev>开发，commit之后，rebase baseline

```
git rebase baseline
```

7. 有冲突的话解决冲突，没有冲突的话，push到origin <local_dev>

```
git push origin local_dev
```

8. 在远程提交Merge Request，done。



### 合并多个commit

1. git rebase -i 到要修改的分支的前一条分支
2. 将pick改成squash，会自动向前合并，如果要修改的话，改成edit。
3. 修改commit的标题：git commit -amend



### 取出另一个分支的某一个commit

```
git cherry-pick <相应的commitID>
```



### 修改commit中作者

```
git commit --amend --author="NewAuthor <NewEmail@address.com>"
```


>>>>>>> 68e8ed34c3381c919d74e33060d87f87d80ef99a


### Git命令详解

#### 添加操作

```shell
git add .                              //全部提交到暂存区
```

#### 提交操作

```shell
git commit -m <description>           //提交到本地库(必须先add)
git commit -am                        //可提交未add文件，但是不包括未创建文件
git commit --amend                    //这个命令会将暂存区中的文件提交。 如果自上次提交以来你还未做任何修改(例如，在上次提交后马上执行了此命令)，那么快照会保持不变，而你所修改的只是提交信息。
```

#### 删除操作

```shell
git rm <file>             //从暂存区删除（stage)  

git rm -f <file>          //删除之前修改过并且已经放到暂存区域
git rm --cached <file>    //如果把文件从暂存区域移除，但仍然希望保留在当前工作目录中，换句话说，仅是从跟踪清单中删除
```

#### 撤销操作

在Git中，用HEAD表示当前版本。

```shell
git HEAD
git HEAD~        //上一个版本
git HEAD~100     //往上100个版本
```
#### 撤销add

```shell
git checkout <file>          //恢复未提交的更改
git reset HEAD <file>        //取消之前 git add 添加
```

#### 撤销commit



![image.png](https://upload-images.jianshu.io/upload_images/5901041-f597c48321c00dcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- git reset

```shell
git reset --hard HEAD~              //回退到上一个版本
git reset --hard <commit ID>        //回退到指定版本
```
版本直接回退，简单粗暴。

如果远程分支也想要回退，git push -f (known changes)。

- git revert

```shell
git revert HEAD //撤销前一次commit
```

不能随便删除已经发布的提交，这时需要通过revert创建要否定的提交。

![image.png](https://upload-images.jianshu.io/upload_images/5901041-384670b871f77b9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果不小心提交了不想要的代码，而小伙伴在你发现时，已经提交了，这时候就不能简单的回退版本。

```shell
git revert <commit ID> //需要撤销的提交ID
```

这时候会有冲突，解决冲突之后，再重新提交，那么就会产生一条新的提交纪录。

提交到远程分支，git push。


> git revert 和 git reset的区别
> - git revert是用一次新的commit来回滚之前的commit，git reset是直接删除指定的commit。
> - 在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为git revert是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是git reset是之间把某些commit在某个branch上删除，因而和老的branch再次merge时，这些被回滚的commit应该还会被引入。
> - git reset 是把HEAD向后移动了一下，而git revert是HEAD继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容。

#### 移动或重命名操作

git mv 命令用于移动或重命名一个文件、目录。

```shell
git mv <file> 
git mv <old name>  <new name>
```

其实，运行 git mv 就相当于运行了下面三条命令：

```shell
mv README.md README
git rm README.md
git add README
```

#### git rebase

##### git rebase和git merge区别

![image.png](https://upload-images.jianshu.io/upload_images/5901041-8914eeaf0da031f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/5901041-75f0907725a60113.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在rebase的过程中，也许会出现冲突(conflict)。 在这种情况，Git会停止rebase并会让你去解决冲突；在解决完冲突后，用"git-add"命令去更新这些内容的索引(index)， 然后，你无需执行 git-commit，只要执行:

```shell
git rebase --continue      //继续
git rebase --abort         //取消
```
##### git rebase -i

在rebase指定i选项，您可以改写、替换、删除或合并提交。

```shell
git rebase -i [startpoint] [endpoint]
```

其中-i的意思是--interactive，即弹出交互式的界面让用户编辑完成合并操作，[startpoint] [endpoint]则指定了一个编辑区间，如果不指定[endpoint]，则该区间的终点默认是当前分支HEAD所指向的commit(注：该区间指定的是一个前开后闭的区间)。

- pick：保留该commit（缩写:p）
- reword：保留该commit，但我需要修改该commit的注释（缩写:r）
- edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
- squash：将该commit和前一个commit合并（缩写:s）
- fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
- exec：执行shell命令（缩写:x）
- drop：我要丢弃该commit（缩写:d）

##### 合并历史纪录

```shell
git rebase -i HEAD~2

//我们会进入vit模式，将pick改成squash，然后按esc : wq（保存并退出）。

git push -f
```

![image.png](https://upload-images.jianshu.io/upload_images/5901041-59881abcdfb62e69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/5901041-92a40a4a997f850d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### git status

要查看哪些文件处于什么状态。

```shell
git status -s | git status --short //得到一种更为紧凑的格式输出
```

#### git diff

git diff 命令显示add与commit的改动区别。

```shell
git diff  <file>            //尚未缓存的改动
git diff --cached           //查看已缓存的改动
git diff HEAD               //查看已缓存的与未缓存的所有改动
git diff --stat             //显示摘要而非整个 diff
```

#### 查看提交历史

##### git log

在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。 完成这个任务最简单而又有效的工具是 git log 命令。

```shell
git log -p          //用来显示每次提交的内容差异
git log -2          //仅显示最近两次提交
git log --stat      //每次提交的简略的统计信息
git log --pretty    //指定使用不同于默认格式的方式展示提交历史，git log --pretty=oneline
```

> 使用git show命令查看某一次提交详细信息。

##### git reflog

> 如果在回退以后又想再次回到之前的版本，git reflog 可以查看所有分支的所有操作记录（包括commit和reset的操作），包括已经被删除的commit记录，git log则不能察看已经删除了的commit记录。

```shell
git reflog
```

#### git stash

在Git中，隐藏操作将使您能够修改跟踪文件，阶段更改，并将其保存在一系列未完成的更改中，并可以随时重新应用。

```shell
git stash          //把当前工作的改变隐藏起来
git stash list     //查看已存在更改的列表
git stash pop      //可从堆栈中删除更改并将其放置在当前工作目录中
```

#### 分支操作

##### 创建分支

```shell
git branch <branch name>               //创建分支
git checkout <branch name>             //切换到分支

git checkout -b <branch name>          //创建并切换到分支
```

##### 删除分支

```shell
git branch -d <branch name>
git branch -D <branch name>       //强制删除分支
```

###### 查看分支

```shell
git branch <name>
git branch -a      //查看所有分支
git branch -r      //查看远程分支
```

##### 重命名分支

```shell
git branch -m <old name> <new name>
```

##### 合并分支

```shell
git checkout master                    //切换到master
git merge <branch name>                //合并分支
```

如果分支未pull最新代码，那么提交的时候，历史纪录就不清晰;汇合分支上的提交，然后一同合并到分支

![image.png](https://upload-images.jianshu.io/upload_images/5901041-2aea300c88e64d2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```shell
git merge –squash <branch name>
git commit -am
git push
```

##### 提取其他分支提交

在cherry-pick，您可以从其他分支复制指定的提交，然后导入到现在的分支。

```shell
git cherry-pick <commit id>
```

#### 标签操作

如果你达到一个重要的阶段，并希望永远记住那个特别的提交快照，你可以使用 git tag 给它打上标签。

##### 创建标签

```shell
git tag <name>
git tag -a <name>              //创建一个带注解的标签
```

##### 查看标签

```shell
git taggit show <tag name>
git push origin <tag name>
```

##### 删除标签

```shell
git tag -d <name>
````

## Progit 第二版 -Scott Chacon



### 基础

#### 优势

* 直接记录快照，而非差异比较。
  * 每次提交更新，或在Git中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。如果文件没有修改，Git不再重新存储该文件，而是只保留一个链接指向之前存储的文件，Git对待数据更像是一个**快照流**。
* 近乎所有的操作都是本地执行。
  * 如果要浏览项目的历史，Git只需从本地数据库中读取。如果想查看当前版本与一个月前的版本之间引入的修改，Git会查找到一个月前的文件做一次本地的差异计算。
* 保证完整性
  * Git中所有的数据在存储前都计算校验和(SHA-1散列)，然后以校验和来引用。Git数据库中保存的信息都是以文件内容的哈希值来索引。
* 一般只添加数据
  * 一旦提交快照到Git中，就很难再丢失数据。

#### 三种状态

1. 已提交 (*committed*)
   * 表示数据已经安全的保存再本地数据库中。
2. 已修改 (*modified*)
   * 表示修改了文件，但是还没保存到数据库中。
3. 已暂存 (*staged*)
   * 对一个已修改文件的当前版本做了标记，使之包含再下次提交的快照中。

#### 跟踪文件

* 工作目录下的文件只有两种状态：
  * 已跟踪(_tracked_)。
  * 未跟踪(_untracked)。

#### <a name="Git是如何保存对象的">Git是如何保存对象的</a>

> 提交操作时，Git会保存一个提交对象(_commit object_)。
>
> 该提交对象包含一个指向暂存内容快照的指针、作者的姓名、邮箱、提交时的输入信息、以及指向它的父对象的指针。
>
> 首次提交产生的提交对象没有父对象，普通提交产生的提交对象有一个父对象，合并产生的提交对象有多个父对象。

> 当使用 git commit 提交时。
>
> Git会先计算每个子目录的校验和，然后在Git仓库中这些校验和保存为树对象。
>
> 随后，Git会创建一个提交对象，它包含了上面提到的信息外，还包含指向这个树对象的指针。

![首次提交对象及其树结构。](https://www.progit.cn/images/commit-and-tree.png)

#### 三棵树

> "树"在这里的实际意思是"文件的集合"，而不是指特定的数据结构。

|        树         | 用途                                 |
| :---------------: | :----------------------------------- |
|       HEAD        | 上一次提交的快照，下一次提交的父节点 |
|       Index       | 预期的下一次提交的快照               |
| Working Directory | 沙盒                                 |

* Head

  > Head 是当前分支引用的指针，总是指向该分支上的最后一次提交。
  >
  > 这表示HEAD将是下一次提交的父节点。
  >
  > 可以将它看做你的**上一次提交**的快照。

  ```bash
  # 查看HEAD快照实际的目录列表，以及其中每个文件的SHA-1的校验和。
  git cat-file -p HEAD
  ```

  

* Index

  > 索引是你的**预期的下一次提交**。
  >
  > 也将这个概念引用为Git的'暂存区域'，也就是git commmit时Git看起来的样子。

  ```bash
  # 显示索引当前的信息
  git ls-files -s
  ```

  

* Working Directory

  > 另外两棵树以一种高效但不直观的方式将他们的内容存储在.git文件夹中。
  >
  > 工作目录会将它们解包为实际的文件以便编辑。
  >
  > 可以将工作目录视为_沙盒_，在将修改提交到暂存区并记录到历史之前，可以随意更改。

  ```bash
  # 查看工作区
  tree
  ```

  ![Git主要通过操纵这三棵树来以更加连续的状态记录项目的快照](https://www.progit.cn/images/reset-workflow.png)

---



### 配置

#### Config

* Git的三层配置：

  * 系统配置

    ```bash
    # 文件路径：etc/gitconfig
    git config --system --list
    ```

  * 全局配置

    ```bash
    # 文件路径：~/.gitconfig 或者 ~/.config/git/config
    git config --global --list
    ```

  * 本地配置（.git/config）

    ```bash
    # 文件路径：.git/config 
    # 只对当前版本库有效
    ```

  ```bash
  # 修改默认的文本编辑器
  git config --global core.editor [vscode]
  
  # 设置commit模板
  git config --global commit.template [~/.gitmessage.txt]
  
  # Windows中检出代码时，换行转换成回车和换行
  git config --global core.autocrlf true
  # Linux或Mac中，提交时把回车和换行转换成换行，检出时不转换
  git config --global core.autocrlf input
  
  # git可以创建4096长度的文件名，Windows最大是260，解决这个问题：
  git config --global core.longpaths true
  ```

#### .gitattribute

> 通过使用 `属性` ，可以对项目中的文件或目录单独定义不同的合并策略，让Git之岛怎样比较非文本文件，或者让Git在提交或检出前过滤内容。

#### 设置别名
```shell
  git config --global alias.co checkout
```

  #### 取消别名
  ```shell
git config --global unset alias.co
  ```



  ```
  
  

#### 忽略文件

* `.gitignorn` 文件的格式规范：
  * 所有空行或者以 `#` 开头的行都会被Git忽略。
  * 可以使用标准的 `glob`式匹配。
  * 匹配模式可以以 `/` 开头方式递归。
  * 匹配模式可以以 `/` 结尾指定目录。
  * 要忽略指定模式以外的文件或目录，可以在模式前加上 `!` 取反。

#### 远程仓库

​```bash
# 查看远程仓库
git remote show [remote-name]
# 需要读写远程仓库使用的Git保存的简介与其对应的URL
git remote -v
# 重命名远程仓库
# 这也会同时修改远程分支的名字
git remote rename [old-name] [new-name]
# 移除远程仓库
git remote rm [remote-name]
  ```



---



### 分支

#### 分支简介

> Git分支，本质上仅仅是指向[提交对象](#Git是如果保存对象的)的可变指针。

#### 查看分支

``` bash 
# 查看各个分支当前所指的对象
git log --oneline --decorate
# 查看项目分叉历史
git log --oneline --decorate --graph --all
# 使用可视化的合并工具
git mergetool
```

```bash
# 查看所有分支
git branch  
# 查看每个分支的最后一次提交
git branch -v
# 查看所有远程分支
git branch -a
# 已经合并到当前分支的分支
git branch --merged
# 尚未合并到当前分支的分支
git branch --no-merged
```

```bash
# 查看跟踪分支(与远程分支有直接关系的本地分支)
git branch -vv
# 设置跟踪分支
git branch --track [remote-branch-name]
```

``` bash
# 删除分支
git branch -d [branch-name]
# 强制删除分支
git branch -D [branch-name]
# 删除远程分支
# 这个命令从服务器上移除这个指针，Git通常会保留数据一段时间直至GC。
git push origin --delete [branch-name]
```



#### 合并分支

> fast-forward：试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，Git在合并两者的时候，只会简单的将指针向前推移，因为这种情况下的合并没有需要解决的分歧。

> 三方合并：当前分支并不是要合并的分支的直接parent，这时，Git会将三方合并的结果做一个新的快照并自动创建一个新的提交指向它，这被称作一次合并提交，它的特别之处在于不止一个父提交。
>
> 需要指出的是，Git会自行决定选取哪个提交作为最优的共同parent，并以此作为合并的基础。

#### 变基

`**不要对在你的仓库外有副本的分支执行变基**`

> Git中整合来自不同分支的修改主要有两种方法：`merge` 和 `rebase`。
>
> 无论哪种方式整合的最终结果所指向的快照始终是一样的，区别只是提交历史不同。
>
> 可以使用rebase命令将提交到某一分支上的所有修改都移到另一分支上。

> 变基的操作实质上是丢弃一些现有的提交，然后相应的新建一些内容一样但实际上不同的提交。如果已经提交推送到远程仓库，并且其他人已经从仓库拉取提交并进行了后续工作。

* 执行方式：
  1. 先找到两者的共同parent。
  2. 对比当前分支相对于该parent分支的历次提交。
  3. 提取相应的修改文件并存为临时文件。
  4. 将当前分支指向目标分支。
  5. 以此将之前另存为临时文件的修改依序应用

![将 `C4` 中的修改变基到 `C3` 上。](https://www.progit.cn/images/basic-rebase-3.png)



---



### Git命令

#### 查看信息

###### diff

``` bash
# 查看已暂存的将要添加到下次提交里的内容
git diff --staged 
# 检查提交信息中是否有'空白错误'
git diff --check
```

###### log

```bash
# 查看在[branch-2]，但不在[branch-1]的提交列表
git log [branch-1]..[branch-2]
git log ^[branch-1] [branch-2]
git log --not [branch-1] [branch-2]
# 查看被两个引用中一个包含但又不被两者同时包含的提交
git log --left-right [branch-1] [branch-2]
```

#### 提交更新

``` bash
# 将所有已跟踪的文件暂存并一起田炯
git commit -a	
# 修改最近一次提交的commit 或 提交完后有修改了文件再次提交时不想保留上次的commit信息
# ！！！如果已经推送就不要修正它。修正会改变SHA-1校验和，类似于一次小的变基。
git commit --amend
# 修改最近的几次提交信息
# ！！！已经推送的不要修改！ 无论是否修改信息，指定范围内的每个提交都会被重写。
git rebase -i HEAD~[n]
```

#### 移除文件

``` bash 
# 手动删除工作区文件并执行
git rm [file-name]
# 可以使用glob模式(删除log/目录下拓展名为.log的所有文件)
git rm log/\*.log
# 如果删除之前修改过，并已经添加到暂存区，需要使用强制删除(force)，这样的数据不能被Git恢复。
git rm -f [file-name]
```

#### 撤销操作

```bash
# 取消暂存的文件
git reset HEAD [file-name]
# 撤销对文件的修改
# ！！！无法撤回，Git实际上会拷贝另一个文件覆盖它
git checkout -- [file-name]
# 放弃本地工作区修改，指定的远程分支覆盖本地
# !!!危险操作
git reset --hard [branch-name]
```

#### 撤销提交

```shell
# 第一种方式：撤销提交。
# ！！！如果已经推送到远程，并且其他人已经有了这个提交，不要使用！
git reset --hard HEAD~

# 第二种方式：还原提交。
git revert -m 1 HEAD
git revert ^M
git merge [branch-name]
```

#### 储藏与清理

> 可以在一个分支上保存储藏，在另一个分支上应用。

```bash
# 储藏工作区修改
git stash
# 包含为跟踪文件
git stash -u
# 查看储藏的信息
git stash list

# 重新应用储藏的信息
# 默认最近的储藏(stash列表继续保留在栈上)
git stash apply
# 重新应用指定的暂存的信息
git stash apply stash@{[n]}
# 应用储藏(并在栈上移除)
git stash pop
# 移除储藏
# 旧版：git stash drop stash@{[n]}
git stash drop -q [n]
```

#### 搜索

> grep 命令可以方便的从提交历史或工作目录中查找一个字符串或者正则表达式

```bash
# 查找并显示行号
git grep -n [keywords]
# 查找并输出哪些文件包含匹配以及每个文件包含了多少匹配
git grep --count [keywords]
# 查看匹配的行属于哪个方法
git grep -p [keywords]
```

```bash
# 查找指定项是什么时候引入和编辑的
git log -S [keywords]
# 查看代码中一行或一个函数的历史
git log -L :[method-name]:[class-name]
```

#### 重置

> reset 做的第一件事是移动HEAD的指向。

* reset 命令会以特定的顺序重写这三棵树，在指定以下选项是：
  1. 移动HEAD分支的指向（若指定了 --soft，则到此停止）。
  2. 是索引看起来像HEAD（若指定了 --hard，则到此停止）。
  3. 是工作目录看起来像索引。

```bash
# 撤销上一次提交，不改变索引和工作目录。
git reset --soft HEAD
# 撤销上一次提交、索引(取消暂存)
# 相等于回滚到了所有git add和git commit命令执行之前。
git reset --mixed HEAD
# 撤销上一次提交、索引以及工作区中的所有工作
# ！！！这是Git中仅有的几个会真正的销毁数据的操作之一！如果Git数据库中的一个提交内还留有修改的文件，可以通过reflog试图找回。
git reset --hard HEAD
```

```bash
# 取消暂存指定的文件(该操作在'三棵树'中的表现形式正好跟git add相反)
git reset --mixed HEAD [file-name]
```

```bash
# 压缩提交
# 将最近的几次提交合并
# git reset将HEAD分支移动到一个旧一点的提交上(即你想要保留的第一个提交)，然后再次git commit。
1 git reset --soft HEAD~[n]
2 git commit 
```

#### 检出

> git checkout [branch-name] 和 git reset --hard [branch-name]非常相似，但是有两点重要区别：

1. checkout 对工作目录是安全的，他会通过检查来确保不会将已更改的文件吹走。它还会在工作目录中先试着简单合并一下，这样还未修改过的文件都会被更新。而reset --hard不做检查就全面的替换了所有东西。

2. reset会移动HEAD分支的指向，而checkout只会移动HEAD自身来指向另一个分支。


#### 合并

```bash
# 退出合并
# 尝试恢复你运行合并前的状态。但当运行命令前，在工作目录中有未储藏、未提交的修改时它不能完美处理。
git merge --abort
# 忽略任意数量的已有空白的修改
git merge -Xignore-space-change [branch-name]
# 忽略所有的空白修改
git merge -Xignore-all-change
# 查看合并引入了什么
git diff --ours
```

###### 手动解决文件冲突再合并

```bash
# 手动解决文件冲突再合并
# 1.获取冲突文件的三个副本:共同的parent版本(common)、你的版本(ours)、合并入的版本(theirs)
# 2.选择其中一个文件进行修复，并为这个单独的文件重试一次合并(git merge-file)
# 3.使用git clean清理不再有用的额外文件
git show :1:program.cs > program.common.cs
git show :2:program.cs > program.ours.cs
git show :3:program.cs > program.theirs.cs

git merge-file -p \
	program.common.cs program.ours.cs > program.cs program.theirs.cs

git clean -f
removing program.common.cs
removing program.outs.cs
removing program.theirs.cs
```

###### 检出冲突

``` bash
# 重新检出文件并替换合并冲突标记
git checkout --conflict=diff3 [file-name]
```

###### 合并日志

```bash
# 查看此次合并中包含的每一个分支的所有独立提交的列表
git log --oneline --left-right HEAD...MERGE_HEAD
# 只显示任何一边接触了合并冲突文件的提交
git log --oneline --left-right --merge
```

###### 其他类型的合并

```bash
# 指定Git遇到冲突时直接选择某一方的文件
git merge -Xours [branch-name]
git merge -Xtheirs [branch-name]
git merge-file --ours
git merge-file --theirs
```

#### 调试

```bash
# 查看指定文件、指定行的修改记录
# ^符号修饰的SHA-1值表示改行是首次提交。
git blame -L 12,22 program.cs
# 找出原始记录
git blame -C -L 12,22 program.cs
```

```bash
# 二分查找
# 最新的代码有Bug，但不知道哪个分支引入的，可以使用二分查找方便的找出
git bisect start 		# 启动
git bisect bad			# 告诉系统当前所在的提交有问题
git bisect good v1.0	# 告诉bisect已知的最后一次正常状态是哪次提交
git bisect good			# 在当前检出的分支上测试后发现没有问题，证明问题是在这个提交之前引入的，告诉git，继续寻找
git bisect bad			# 重新检出分支后发现问题存在
git bisect reset 		# 完成操作之后，重置HEAD指针到最开始的位置。
```

#### 属性

```bash 
# 比较不同版本的Word文件
# 将下面的文本添加到 .gitattributes 文件中：
# *.docx diff=word
```



---



### 附录

#### git status -s 状态码

| 状态码 |                说明                |
| :----: | :--------------------------------: |
|   ??   |         新添加的未跟踪文件         |
|   A    |        新添加到暂存区的文件        |
|  M__   |      该文件被修改并放入暂存区      |
|  __M   |    该文件被修改还没有放入暂存区    |
|   MM   | 该文件在暂存区和工作区都有修改记录 |
|   D    |        该文件在工作区被删除        |
|   C    |          文件的一个新拷贝          |
|   R    |            文件名被修改            |
|   T    |           文件类型被修改           |
|   U    |           文件没有被合并           |
|   X    |              未知状态              |

#### checkout 和 set 命令对三棵树的影响

> `REF` 表示该命令移动了HEAD指向的分支引用，`HEAD` 则表示只移动了HEAD自身。

|         command          | HEAD | Index | Workdir | WD Safe? |
| :----------------------: | :--: | :---: | :-----: | :------: |
|     **Commit Level**     |      |       |         |          |
|  reset --soft [commit]   | REF  |  NO   |   NO    |   YES    |
|      reset [commit]      | REF  |  YES  |   NO    |   YES    |
|   reset -hard [commit]   | REF  |  YES  |   YES   |  **NO**  |
|    checkout [commit]     | HEAD |  YES  |   YES   | YESFile  |
|      **File Level**      |      |       |         |          |
|  reset (commit) [file]   |  No  |  YES  |   NO    |   YES    |
| checkout (commit) [file] |  NO  |  YES  |   YES   |  **NO**  |










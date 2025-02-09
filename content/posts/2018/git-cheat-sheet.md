---
title: "Git Cheat Sheet"
date: 2018-03-11T13:07:40
categories:
- Others
tags:
- Cheat sheet
- Git
---

## 本地操作 ##

### 状态检览 ###

```shell
$ git status -s
XY PATH1 -> PATH2
```
PATH2只有在PATH1关联到不同的路径时才会显示(例如, 文件重命名).
<!-- more -->
XY是两个状态码, 在合并冲突的时候, X和Y分别表示合并双方的修改状态; 而在一般情况下X表示暂存区域(_index_)的状态, Y表示工作目录的状态(_work tree_):
-   ' ' = unmodified
-   M = modified
-   A = added
-   D = deleted
-   R = renamed
-   C = copied
-   U = updated but unmerged

未被追踪(_untracked_)的文件, XY = ??; 默认不显示忽略的文件(_ignored_), 除非使用`--ignore`选项, 此时XY = !!.

### 查看已暂存和未暂存的修改 ###
```shell
$ git diff
```
用来比较工作目录中当前文件和暂存区域快照之间的差异, 使用`--staged`选项查看已经暂存的将要添加到下次修改的内容

### 提交更新 ###
```shell
$ git commit
```
这种方式会启动shell的环境变量`$EDDITOR`所指定的软件, 一般是VIM或emacs, 或者使用`git	config	--global	core.editor`来指定编辑器,
使用`-v`选项将`diff`的内容追加到编辑器中, 使用`-m '${comment}'`选项来直接添加提交信息, 而不打开编辑器.

#### 跳过使用暂存区 ####
```shell
$ git commit -a
```
git会自动把所有已经跟踪过的文件暂存起来一并提交, 跳过`git add`步骤.

### 移除文件 ###
```shell
$ git rm ${fileName}
```
移除工作目录文件, 并记录移除操作, 如果删除之前修改过并且已经放到暂存区域的话, 必须使用强制删除选项`-f`, 用于防止误删.

```shell
$ git rm --cached ${fileName}
```
从git仓库(亦从暂存区移除)移除文件, 而保留磁盘文件

### 移动文件 ###
```shell
$ git mv ${oldFileName} ${newFileName}
```
等价于
```shell
$ mv ${oldFileName} ${newFileName}
$ git rm ${oldFileName}
$ git add ${newFileName}
```

### 查看提交历史 ###
```shell
$ git log
```
默认情况下, git会按提交时间列出所有的更新, 最近的更新排在最上面. 使用`-p`选项来显示每次提交的内容差异; `-${n}`选项来明确要显示几次提交,
n为自然数; `--stat`选项输出每次提交的简略统计信息; `--pretty`来指定使用不同的默认格式的方式展示提交历史, 格式有`oneline`, `short`,
`full`和`fuller`, 或者用`--pretty=format:"${format}"`来指定显示格式; `--graph`选项使用ASCII字符形象地展示分支, 合并历史.

### 撤销操作 ###
```shell
$ git commit --amend
```
用来尝试重新提交, 修改提交信息.
```shell
$ git reset HEAD ${fileName}
```
撤销文件暂存.
```shell
$ git checkout -- ${fileName}
```
撤销对文件的修改(丢失修改信息).

## 远程仓库使用 ##

### 查看远程仓库 ###
```shell
$ git remote
```
使用`-v`选项来显示对应的url.

如果想要查看一个远程仓库的更多信息, 使用`git remote show ${remoteName}`

### 从远程仓库中抓取与拉取 ###
```shell
$ git fetch ${remoteName}
```
访问远程仓库, 从中拉取所有你还没有的数据, 而不会自动合并或修改当前的工作, 需要手动合并.

使用`clone`会自动将其添加为远程仓库并默认为'origin'.

如果一个分支设置为跟踪一个远程分支, 可以使用`git pull`来自动抓取然后合并远程分支到当前分支; 默认情况`git clone`会自动的设置本地master
分支跟踪clone的远程仓库的master分支

### 远程仓库的移除与重命名 ###
```shell
git remote rename ${oldName} ${newName}
git remote rm ${remoteName}
```


## 分支管理 ##

### 分支的新建与合并 ##

```shell
$ git checkout -b ${branchName}
```
创建新分支并切换当前分支至新分支, 等价于:
```shell
$ git branch ${branchName}
$ git checkout ${branchName}
```
```shell
git merge ${branchName}
```
合并到当前分支, 使用`rebase`来确保在向远程分支推送时提交历史的整洁, 即`rebase`和`merge`在提交历史上的存在差异.

### 分支管理 ###
```shell
git branch
```
显示所有分支, `-v`选项来查看每一个分支的最后一次提交信息; `--merged`和`--no-merged`来查看已经合并或尚未合并到当前分支的的分支; `-d`选项来
删除已经完全合并了的分支, `-D`选项强制删除分支.

## 配置 ##
```shell
git config credential.helper store
```
保存使用HTTP协议的密码

### 换行符 ###
由于文本文件使用的换行符在不同的操作系统下不一致的问题(在Unix/Linux上为`0x0A(LF)`, 而在DOS/Windows上为`0x0D0A(CRLF)`), 当换行符发生改变时,
Git会认为整个文件被修改而导致无法`diff`, 此时一种解决方案是设置全局的`autocrlf`配置项:
```shell
git config --global core.autocrlf true
```
其中配置项的值可以是:
1. __true__: 提交时转为LF, 检出时转为CRLF
2. __false__: 提交检出均不转换
3. __input__: 提交时转为LF, 检出时不转换

另外还有`safecrlf`配置项, 其值为:
1. __true__: 拒绝包含混合换行符文件的提交
2. __false__: 允许包含混合换行符文件提交
3. __warn__: 提交包含混合换行符文件时给出警告

另外一种方案则是对每个仓库单独建一个`.gitattributes`文件来单独配置:
```
# from https://help.github.com/articles/dealing-with-line-endings/
# Set the default behavior, in case people don't have core.autocrlf set.
* text=auto

# Explicitly declare text files you want to always be normalized and converted
# to native line endings on checkout.
*.c text
*.h text

# Declare files that will always have CRLF line endings on checkout.
*.sln text eol=crlf

# Denote all files that are truly binary and should not be modified.
*.png binary
*.jpg binary
```
其中几条规则:
1. `text=auto`: Git自动处理这些文件, 通常建议的默认选项
2. `text eol=crlf`: Git将在检出这些文件时自动转成CRLF, 建议将一些必须CRLF的文件设为该配置(如`.bat`), 另外还有`eol=lf`
3. `binary`: Git会认为这些文件不应该改变, 等同于`text -diff`

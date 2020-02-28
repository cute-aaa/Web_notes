# Git笔记

git init '[dir]'指定目录（单引号）

git add 将想要快照的文件加入缓存区（每次更新文件时也要先添加）

- git add . 添加所有

git commit  将缓存区内容添加到**本地仓库**

- git commit -a 提交所有，可省去add操作
- git commit -m 提交文件的描述，如文件哪里有改动
- git commit -am "描述" 直接提交全部修改

git rm <file> 从工作目录中删除文件

- git rm --cached <file> 从同步列表中删除而不删除源文件

- git rm -f <file> 删除之前修改过并且已经放到暂存区的文件（强制删除）
- git rm -r * 递归删除

git mv 重命名或移动文件、目录、软连接

- git mv readme readme.md

git push 将本地库的最新信息发送给**远程库**

git pull 从**远程库**获取最新版本到本地并自动merge

- 相当于git fetch + git merge

git fetch 从远程库获取最新版本到本地，不自动merge

git merge 用于从指定的commit(s)合并到当前分支，用来合并两个分支（merge：合并）

- git merge -b 将b分支合并到当前分支

git reset HEAD [文件名] 在上传列表中删除文件使其不会被上传

git status 查看在上次提交之后是否有修改

git status -s 查看目录下文件状态

- ??表示未加入同步
- A表示同步完成
- AM表示有改动

git diff 查看执行git status 的结果的详细信息，如具体改动的内容

- git diff  尚未缓存的改动
- git diff -- cached  查看已缓存的改动
- git diff HEAD  查看已缓存的与未缓存的所有改动
- git diff --stat 显示摘要

git clone <source repository> <destination repository> 复制本地方式：本地仓库，想克隆去另一个地方的路径

- git clone d:/git e:/git 将d:/git的仓库（包含.git的目录）克隆到e:/git下
- 目标目录必须为创建或者为空
- 与从远程拉取仓库不同，路径的最后不用写.git来表明这是一个仓库



#### 分支管理

从开发主线上分离开，然后在不影响主线的同时继续工作

git branch 列出分支，带*的是当前分支

git branch (branchname) 创建分支

- git branch test 创建名为test的分支

git checkout (branchname) 切换分支

- 切换分支时git会用该分支的最后提交的快照替换你的工作目录的内容，所以多个分支不需要多个目录，而是都在同一个目录

git merge 合并分支。可以多次合并到统一分支，也可以在合并之后直接删除被并入的分支



### 设置

git config --global user.name 'sure'

git config --global user.email sure@2316757392@qq.com



当更改git所在位置时，会提示找不到证书文件

**解决方法**

git config --system http.sslcainfo "C:\Program Files (x86)\git\bin\curl-ca-bundle.crt"
（注意修改为正确的文件路径）或

git config --system http.sslverify false
---
title: 使用git基础命令
categories:
  - Git
tags:
  - git
toc: true # 是否启用内容索引
---

##### 1.使用Git拉去Fork分支步骤：

1.打开到GOPATH目录下，右键`Git Bash Here`,进入Git命令界面（所有部门代码都是在navi文件夹下打开，F:\vagrant\data\gopath\src\navi）

2.克隆远程仓库到本地

```
git clone <url> 	 //url:fork分支后的http地址
```

3.进入到 clone 的目录（F:\vagrant\data\gopath\src\navi\XXXX文件夹）

```
cd ..
```

3.关联远程仓库

```
git remote add main <url>  //url:navi仓库http地址
```

4.查看远程仓库

```
git remote -v
```

5.拉取远端main所有分支到本地

```
git fetch main
```

6.查看拉取到本地的分支列表

```
git branch -a
```

7.创建分支 （一般是develop）

```
git branch <name> main/<name>
```

8.切换分支

```
git checkout <name> 
```

或者步骤7和8合并：创建并切换 

```
 git checkout -b <name>  main/<name>
```



查看远程仓库信息

```shell
git remote
```

查看远程仓库详细信息

```shell
git remote -v
```

与远程仓库代码同步

```shell
git pull
# git pull = git fetch + git merge
```

**如果你想完全地覆盖本地的代码，只保留服务器端代码，则直接回退到上一个版本，再进行pull**

```
git reset --hard
git pull
```

其中git reset是针对版本,如果想针对文件回退本地修改,使用

```
git checkout HEAD [file]
```

##### 2.提交代码到远程仓库新项目

1.关联origin远程仓库

```
git remote add origin http://yanwq@192.168.1.189/yanwq/koe.git
```

2.关联main远程仓库

```
git remote add main http://yanwq@192.168.1.189/navi/koe.git

```

3.查看远程仓库详细信息

```shell
git remote -v
```

4.拉取main远程仓库 develop分支

```
git pull mian develop

//出现报错
fatal: refusing to merge unrelated histories
```

5.拉取main远程仓库develop分支 历史不相关代码

```
git pull --allow-unrelated-histories main develop

//--allow-unrelated-histories不相关拉取
//出现报错
error: Your local changes to the following files would be overwritten by merge:
```

6.git工作区状态：

```
git status

//出现绿色的
Changes to be committed:
	new file :XXX
	deleted: XXX
	renamed:XXX
//出现红色的	
Changes not staged for commit:
	modified:XXX
```

7.把文件添加进去，实际上就是把文件修改添加到暂存区；

```
git add .

//warning: LF will be .....
```

8.git工作区状态：

```
git status
//只有绿色
```

9.`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。 “feat:init"实际就是远程仓库的Last commit 字段，一般为写入提交更改的具体模块

```
git commit -m "feat: init"
```

10.git工作区状态：

```
git status 
//出现
On branch develop
nothing to commit, working tree clean
```

11.把两段不相干的 分支进行强行合并

```
git pull --allow-unrelated-histories main develop
```

12.推送提交到远程仓库（自己分支）

```
git push origin develop
```

13.推送提交到远程仓库

```
git push main develop
```

14.查看详细提交历史

```
git log

//q退出
```





##### 3.修改远程仓库已有的项目代码

1.查看远程仓库详细信息

```
git remote -v
```

2.查看git工作区状态

```
git status

//出现绿色的
Changes to be committed:
	new file :XXX
	deleted: XXX
	renamed:XXX
//出现红色的	
Changes not staged for commit:
	modified:XXX
```

3.把文件添加进去，实际上就是把文件修改添加到暂存区；

```
git add .

//warning: LF will be .....
```

4.git工作区状态：

```
git status
//只有绿色
```

5.`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。 “feat:init"实际就是远程仓库的Last commit 字段，一般为写入提交更改的具体模块

```
git commit -m "feat: init"
```

6.git工作区状态：

```
git status 
//出现
On branch develop
nothing to commit, working tree clean
```

7.推送提交到远程仓库（只需要origin）

```
git push         //origin develop
```

8.ctril点击 第7步返回的remote ： http://.... ，跳转地址，点击

```
Change branches
```

9.点击合并分支并继续

```
Compare branches and continue
```

10.接受合并请求

```
Accept merge request
```





问题：有时候遇到需要删除本地代码，从远程仓库中拉取最新代码的情况。
解决方法：当存在多个分支时，首先切换到当前的分支

```javascript
git checkout master
```

然后 git 强行pull并覆盖本地文件 （一般到develop分支）

```javascript
git fetch --all  
git reset --hard origin/master 
git pull
```





#### 方法一 通过命令直接修改远程地址

1. 进入git_test根目录
2. git remote 查看所有远程仓库， git remote xxx 查看指定远程仓库地址
3. git remote set-url origin http://192.168.100.235:9797/john/git_test.git
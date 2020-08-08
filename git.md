## GIT简介
​        Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。



## GIT使用
~~~bash
git init	//创建仓库

git add -A	//添加所有变化

git commit -m "message"		//提交，其中message是说明

git status 		//查看仓库状态

git branch		//查看分支，当前分支前面有*

git branch dev	//创建dev分支

git checkout dev	//切换dev分支

git checkout -b dev //创建dev分支，并切换

git branch -d dev 	//删除dev分支

git merge dev	//合并dev分支到当前分支


git remote 	//查看远程信息

git remote add origin git@github.com:doc/abc.git	//关联远程仓库
 
git push -u origin master	//第一次推送到远程仓库，需要-u参数

git clone git@github.com:doc/abc.git	//从远程克隆

git pull 	//从远程获取合并到本地

git branch --set-upstream-to <branch-name> origin/<branch-name>
//建立本地分支和远程分支的链接关系



~~~
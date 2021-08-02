## GIT简介
​        Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。



> git 分为工作区（work）、缓存区（index），和仓库区（repository）



## GIT使用
~~~bash
git init	//创建仓库

git add -A	//添加所有变化到缓存区

git commit -m "message"		//提交，其中message是说明，提交到仓库

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

git checkout # 拉取缓存替换工作区文件 可加`--<file_name>`指定文件或使用.表示所有文件

## 如果想撤销某次commit(reset 分为soft和hard，默认为soft)
# 回退到上一次提交之前
git reset HEAD^ #从最近一次提交拉取文件覆盖缓存区，工作区不变，可加`--<file_name>`指定文件

git reset --hard HEAD^ #从最近一次提交拉取文件覆盖缓存区和工作区，代码文件全部变成上一次提交的文件，可加`--<file_name>`指定文件

# 回退到任意一次提交
git reset <commit_id> 	# <commit_id>可以使用git log获取



# 重新add和commit后，再推送到github需要加`-f`参数
git push -f

# 如果提交后再添加`.gitignore`,而github上还在，先清除缓存再提交
git rm -r --cached .
# 重新add和commit后，再推送到github需要加`-f`参数
git push -f


# 如果本地和远程出现冲突（Please, commit your changes or stash them before you can merge.
）
git stash	# 将本地修改存入缓存并恢复一次提交
git pull	# 远程拉取
git stash pop	# 恢复缓存内容
~~~
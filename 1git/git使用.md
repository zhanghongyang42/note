项目中使用git构建分支流四种方式：	https://www.cnblogs.com/quartz/p/12814064.html

教程：https://www.liaoxuefeng.com/wiki/896043488029600/900388704535136



# 概念

![1543302039138](picture/1543302039138.png)



### 本地仓库

本地仓库：狭义的本地仓库特指版本库

​					广义的本地仓库包括 版本库，工作区，暂存区。



版本库（）：.git文件夹。这里的版本就是一个点，可以认为你每次commit都是一个版本。里面包括暂存区，master分支，指针HEAD

工作区：包含.git文件夹的目录，即可视的文件都位于工作区

暂存区：工作区到版本库的一个缓冲区域，位于.git中，工作区多次add到暂存区



创建本地仓库

git bash   ：     git   init



### 远程仓库

远程仓库：github、gitlib、码云、自建远程仓库

github使用：	https://www.zhihu.com/question/21669554



### 忽略文件

忽略文件不保存到版本库

在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去

```
*.a
!lib.a
/TODO
build/
```



# 文件保存

**工作区文件**

将文件放到工作区即可



**工作区添加暂存区**

git add readme.txt



**暂存区提交本地仓库**

git commit -m “注释”



# 分支操作

查看分支

git branch



创建分支

git branch dev



切换分支

git checkout dev



删除分支

git branch –d name



合并分支，将dev分支合并到当前分支上

git merge  –no-ff  dev



合并分支冲突

手动解决分支冲突再合并



# 工作区暂存

工作到一半，需要切换到其他分支工作，但是还不想commit即可暂存



1.暂存

git bash   ：     git stash

暂存后即可切换分支，修复bug后，切换回本分支来恢复



2.查看

git bash   ：     git stash list



3.恢复

git bash   ：     git stash pop



# 版本回退

**工作区修改回退**

工作区回退到版本库版本：做了修改，还没add，回到版本初始状态。

git checkout --文件名称



工作区回退到暂存区版本：add到暂存区了，又做了修改，想恢复暂存区文件。

git checkout --文件名称



**暂存区修改回退**

清空暂存区某些文件

git reset HEAD file



工作区回退到版本库版本

git checkout --文件名称



**版本回退**

版本库会维护所有commit过的版本，每个版本都包括暂存区+工作区分支。

版本回退，会同时改变 暂存区+工作区。



查看版本

git reflog 



回退到指定版本号

git reset --hard 版本号 



# -----------------------



# 远程仓库

可以自建远程仓库服务器，也可以使用github，gitlab等远程仓库。

一般的远程仓库提供ssh和http两种方式连接交互，ssh需要生成ssh密钥，http需要账号密码。

下面我们以 github 为例使用远程仓库。



查看远程仓库

git remote

git remote -v



# 移除远程仓库

git remote rm origin



# 本地克隆到远程

一般不会采用这种方法，采用远程克隆到本地



建一个本地仓库

建一个远程仓库

![v2-044b0f2143423da8ead81e8cdd93cf92_r](picture/v2-044b0f2143423da8ead81e8cdd93cf92_r.jpg)



将本地仓库克隆到远程库

git remote add origin https://github.com/zhanghongyang42/远程仓库名.git



将本地分支克隆到远程

 git push -u origin master



# 远程克隆到本地

建一个远程仓库



将远程仓库克隆到本地仓库

git clone https://github.com/zhanghongyang42/远程仓库名



首先要保证有本地的git账号和git邮箱。

克隆的两种方式， http和ssh

http 方式

```
 清除gitlab或者github的账户名和密码
 git credential-manager uninstall
 
 需要的时候要输入 gitlab或者github的账户名和密码
 
  永久记住密码
 git config credential.helper store
```

ssh 方式，需要为每个项目配置公钥，暂时省略



# 本地远程分支映射

查看映射关系

git branch -vv



保证本地与远程都有分支，建立本地与远程分支的映射关系

 git branch --set-upstream-to origin/dev dev



# 拉取分支

第一次需要创建并切换本地分支，

建立本地分支与远程分支的映射，

可以简化为

git checkout -b dev origin/dev



就可以无限拉取了。 

git pull



# 推送分支

第一次推动需要建立联系 

git push -u origin master 



以后推送，确保已经建立联系了，master是本地分支名称

git push origin master 



# 远程分支冲突

先把远程冲突分支git pull下来

手动解决冲突

commit 

push



# -----------------------



# 在IDEA中使用git

https://www.cnblogs.com/boonya/p/8185169.html



### 在 idea中配置git

安装好IntelliJ IDEA后，如果Git安装在默认路径下，那么idea会自动找到git的位置，如果更改了Git的安装位置则需要手动配置下Git的路径。

选择File→Settings打开设置窗口，找到Version Control下的git选项：

![1543399359546](picture\1543399359546.png)

选择git的安装目录后可以点击“Test”按钮测试是否正确配置。

![1543399391526](picture\1543399391526.png)

![1550563988719](picture\1550563988719.png)

### 将工程添加到git

* 1) 在idea中创建一个工程, 例如创建一个java工程, 名称为idea-git-test, 如下图所示

![1543399570808](picture\1543399570808.png)

* 2）创建本地仓库
  * 在菜单中选择“vcs”→Import into Version Control→Create Git Repository...

![1543399681865](picture\1543399681865.png)



![1543399818615](picture\1543399818615.png)  

选择之后在工具栏上就多出了git相关工具按钮：

![1543399982814](picture\1543399982814.png)

* 将其添加到本地版本库中: 点击commit即可提交到本地的版本库中

![1543400525142](picture\1543400525142.png)

* 推送至远程

  在码云上创建一个仓库然后将本地仓库推送到远程。

  在工程上点击右键，选择git→Repository→push，

  或者在菜单中选择vcs→git→push

![1543400803227](picture\1543400803227.png)

![1543400828571](picture\1543400828571.png)

> 选择Define remote

![1543400956443](picture\1543400956443.png)

![1543400987266](picture\1543400987266.png)

成功后, idea会显示

![43401045702](picture\1543401045702.png)





添加忽略文件的插件：

![1550565488675](picture\1550565488675.png)

### 从远程仓库克隆

​	关闭工程后，在idea的欢迎页上有“Check out from version control”下拉框，选择git

![1543401467964](picture\1543401467964.png)

![1543401508478](picture\1543401508478.png)



![1543401651635](picture\1543401651635.png)

> 使用idea选择克隆后, 会出现如下内容, 一直下一步即可

![1543401862519](picture\1543401862519.png)

> 此时就又回来了

![1543401893387](picture\1543401893387.png)



# 待整理

git log

查看本地仓库状态 git status

查看工作区文件修改内容 git diff readme.txt

查看版本库修改日志 git log –pretty=oneline

查看用户邮箱 git config --list

git remote
git remote -v







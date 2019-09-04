### 一、基础配置

#### 1、安装git

​	https://git-scm.com/downloads ,这个是地址，自行安装。

​	之后以下命令全部在git Bash中执行

#### 2、配置本地git

~~~
$ git config --global user.name "yanglinAlwork"
$ git config --global user.email "18234176953@163.com"
~~~

#### 3、创建版本库

​	版本库可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

##### 	**1）创建本地目录**

~~~
cd E:
mkdir learngit
cd learngit
~~~

注： 如果你使用Windows系统，为了避免遇到各种莫名其妙的问题，请确保目录名（包括父目录）不包含中文。

##### 	**2）通过git init命令把这个目录变成Git可以管理的仓库：**

~~~
$ git init
Initialized empty Git repository in /Users/michael/learngit/.git/
~~~

##### 	**3）新建一个文件 demo-dev.yml**

##### 	**4）把文件放进版本库**

~~~
git add demo-dev.yml
~~~

##### 	**5）用命令`git commit`告诉Git，把文件提交到仓库：**

~~~
git commit -m "wrote a demo-dev.yml file"
~~~

~~~
[master (root-commit) eb1a70b] wrote a demo-dev.yml file
 1 file changed, 1 insertion(+)
 create mode 100644 demo-dev.yml
 这个是提交日志， eb1a70b这个是id，很重要！！！
~~~

​	简单解释一下`git commit`命令，`-m`后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。

​	为什么Git添加文件需要`add`，`commit`一共两步呢？因为`commit`可以一次提交很多文件，所以你可以多次`add`不同的文件，比如：

```
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."
```

#### 4、常用基础命令

##### 	**1）查看git提交命令**

~~~
git log
~~~

##### 附：查看分支提交详细信息

~~~
git log --graph --pretty=oneline --abbrev-commit
~~~

##### 	**2）回滚git**

~~~
git reset --hard HEAD^
回滚到上一个版本
git reset --hard 1094a
回滚到指定id版本 
~~~

##### 	**3）查看每次提交的id**

~~~
git reflog
~~~

##### 	**4）查看版本控制中是否有未提交的文件**

~~~
git status
~~~

##### 	**5）撤销修改**

```
 git checkout -- demo-dev.yml
```

​	命令`git checkout -- demo-dev.yml`意思就是，把`demo-dev.yml`文件在工作区的修改全部撤销，这里有两种情况：

​	一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

​	一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

​	总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

~~~
git reset HEAD demo-dev.yml
~~~

​	用命令`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区。

##### 	**6）删除文件**

~~~
git rm test.txt
~~~

##### 	**7）恢复错删的本地文件**

~~~
git checkout -- test.txt
~~~

`	git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

---

### 二、添加远程git库

​	注意：有些网络访问不了 git开头的域名，换用https即可

​	github账户

​	**用户名：**18234176953@163.com

​	**密码：**手机号+小写名字+点

​	在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作，真是一举多得。

​	首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库：

![github-create-repo-1](F:\GitDepository\学习文档\Java\baseImages\添加远程git.png)

​	在Repository name填入`learngit`，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库：

#### 1、从本地到git仓库

##### 	1）把本地仓库的内容推送到GitHub仓库。

~~~
git remote add origin git@github.com:yanglinAIwork/learngit.git
~~~

​	请千万注意，把上面的`yanglinAIwork`替换成你自己的GitHub账户名，否则，你在本地关联的就是我的远程库，关联没有问题，但是你以后推送是推不上去的，因为你的SSH Key公钥不在我的账户列表中。

​	添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。

​	**2）把本地库的所有内容推送到远程库上**

~~~
git push -u origin master
~~~

​	由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

​	从现在起，只要本地作了提交，就可以通过命令： master 是工作分支名，如果是dev分支就写 dev

```
$ git push origin master
```

​	推送时提示以下错误的解决方法

​	**错误1：**

~~~
$ git push -u origin master
fatal: 'git@github.com/yanglinAIwork/learngit.git' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
~~~

​	**解决方法：**

~~~
修改“.git”文件夹里面的“config”文件的url就可以了：

原 url = https://github.com/checkfrank/checkfrank.github.io.git
修改为
url = https://github.com/yanglinAIwork/learngit
~~~

​	**错误2：**

~~~
Logon failed, use ctrl+c to cancel basic credential prompt.
Username for 'https://github.com': 18234176953@163.com
remote: Invalid username or password.
fatal: Authentication failed for 'https://github.com/yanglinAIwork/learngit/'
~~~

​	**解决方法：**

~~~
生成ssh key
ssh-keygen -t rsa -C "18234176953@163.com"

一直回车，直至生成id_rsa和id_rsa.pub
用记事本之类的软件打开id_rsa.pub文件，并且复制全部内容
在你的gitlab或者github的账户，打开SSH key标签。
选择Add SSH key按钮，将刚刚复制的内容粘贴进去即可，然后点击add key。
~~~

​	**错误3：**

~~~
fatal: unable to access 'https://github.com/yanglinAIwork/learngit/': Failed to connect to github.com port 443: Timed out
~~~

​	**解决方法：**

~~~
$ git config --global http.proxy
 
$ git config --global --unset http.proxy
~~~

​	**错误4：**

~~~
$ git pull origin master
error: You have not concluded your merge (MERGE_HEAD exists).
hint: Please, commit your changes before merging.
fatal: Exiting because of unfinished merge.
~~~

​	**解决方法：**

~~~
先 commit本地的文件
~~~

#### 2、从git仓库到本地

~~~
git clone git@github.com:yanglinAIwork/learngit.git
~~~

#### 3、查看远程库的信息

~~~
git remote
~~~

或者，用`git remote -v`显示更详细的信息：

~~~
$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
~~~

​	上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址。

---

### 三、git分支

​	分支在实际中有什么用呢？假设你准备开发一个新功能，但是需要两周才能完成，第一周你写了50%的代码，如果立刻提交，由于代码还没写完，不完整的代码库会导致别人不能干活了。如果等代码全部写完再一次提交，又存在丢失每天进度的巨大风险。

​	现在有了分支，就不用怕了。你创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上，这样，既安全，又不影响别人工作。

​	其他版本控制系统如SVN等都有分支管理，但是用过之后你会发现，这些版本控制系统创建和切换分支比蜗牛还慢，简直让人无法忍受，结果分支功能成了摆设，大家都不去用。

​	但Git的分支是与众不同的，无论创建、切换和删除分支，Git在1秒钟之内就能完成！无论你的版本库是1个文件还是1万个文件。

---

#### 1、创建和合并分支

##### 1）创建`dev`分支，然后切换到`dev`分支

~~~
git checkout -b dev
~~~

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev
$ git checkout dev
```

##### 2）查看当前分支

~~~
git branch
~~~

##### 3）切换分支错误及解决方案

~~~
error: pathspec 'branch' did not match any file(s) known to git
~~~

​	这个是因为当前分支是master分支的项目，强行换成dev分支是不可能的，也可以转，自行百度。

##### 4）dev分支的工作

​	对于最初创建的demo-dev.yml（master提交的），可以切换到dev分支进行开发，然后 add commit，无论dev环境下如何add、commit，master的文件都是不变的。之后对分支进行合并。合并操作如下：

~~~
git merge dev
~~~

##### 5）删除分支

~~~
git branch -d <name>
~~~

​	删除未合并的分支，需要使用 git branch -D <name> 强行删除

##### 6）分支冲突

​	如果遇到a、b两个分支同时操作一个文件，合并分支时就会冲突，此时手动修改冲突部分。

​	解决方案：

​	a、先pull远程分支下来（不会把本地修改过的冲掉）

~~~
git pull origin master
~~~

​	b、修改后在push上去

##### 7）查看分支合并图

~~~
git log --graph
~~~

##### 8）开发中临时新增分支

​	如果在开发中dev分支正在开发一半，但临时需要新增一个bug1分支进行紧急任务，在不提交dev分支的情况下，按如下操作：

​	**a.储藏当前工作进度**

~~~
git stash
~~~

​	**b. 查看工作区**

~~~
git status
~~~

​	此时工作区是干净的

​	**c. 在紧急任务的分支上新建bug1分支，修改提交之后删除bug1分支**

​	**d. 恢复dev分支进度**

~~~
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge

先查看进度所在地
然后恢复

git stash pop
~~~

​	注：可以用 git stash apply stash@{0} 进行恢复，但此命令不会删除stash进度的存储，需要手动删除

​	**删除stash进度**

~~~
git stash drop
~~~

---

### 四、版本标签

​	之前一系列回滚操作都需要用到提交的id，比如 commit号是6a5819e，把这个改成版本号就好记了。这就是标签。

#### 1、创建标签

##### 1）切换到需要打标签（版本号）的分支

~~~
$ git branch
* dev
  master
$ git checkout master
Switched to branch 'master'
~~~

##### 2）打标签

~~~
git tag v1.0
~~~

​	默认标签是打在最新提交的commit上的。

​	注：有时候，如果忘了打标签，比如，现在已经是周五了，但应该在这周一打的标签没有打，怎么办？

​	解决方案：

​	a、找到历史提交的comit id

```
git log --pretty=oneline --abbrev-commit
```

​	b、给对应id 打标签

```
git tag v0.9 f52c633
```

##### 附：创建带说明的标签

~~~
还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：

$ git tag -a v0.1 -m "version 0.1 released" 1094adb
~~~

##### 3）查看标签

~~~
git tag
~~~

​	注意：标签不是按时间顺序列出，而是按字母排序的

##### 附：查看标签信息

~~~
git show <tagname>
~~~

---

#### 2、删除标签

##### 1）删除本地标签

~~~
git tag -d v0.1
~~~

​	因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

##### 2）推送标签到远程

~~~
git push origin <tagname>
~~~

或者，一次性推送全部尚未推送到远程的本地标签

~~~
git push origin --tags
~~~

##### 3）删除远程标签

​	**a、先删除本地标签**

~~~
git tag -d v0.9
~~~

​	**b、再删除远程标签**

~~~
git push origin :refs/tags/v0.9
~~~

---

## 附录：

### 1、码云（gitee.com）

​	使用GitHub时，国内的用户经常遇到的问题是访问速度太慢，有时候还会出现无法连接的情况（原因你懂的）。

​	如果我们希望体验Git飞一般的速度，可以使用国内的Git托管服务——[码云](https://gitee.com/)（[gitee.com](https://gitee.com/)）。

 	码云的免费版本也提供私有库功能，只是有5人的成员上限。

​	使用码云和使用GitHub类似，不再过多解释。[详见地址](https://www.liaoxuefeng.com/wiki/896043488029600/1163625339727712)

### 2、忽略特殊文件

​	[详见地址](https://blog.csdn.net/lk142500/article/details/82869018)

​	1）修改.gitignore文件即可。

​	2）检查`.gitignore`文件

~~~
git check-ignore
~~~

​	3）强行添加被忽略的特殊文件到版本库

~~~
git add -f App.class
~~~

### 3、搭建Git服务器

​	[详见地址](https://www.liaoxuefeng.com/wiki/896043488029600/899998870925664)

### 4、利用WebHook实现Github或Coding代码的自动部署

#### 1、准备钩子文件

​	**创建和修改目录权限：**

~~~
#创建目录
mkdir coding-webhook 
~~~

​	**写入钩子文件：**

~~~
Yanglin@DESKTOP-G544I0V MINGW64 /e/GitRepository
$ cd coding-webhook/

Yanglin@DESKTOP-G544I0V MINGW64 /e/GitRepository/coding-webhook
$ touch webhook.php
~~~

​	**钩子文件内容**

~~~
<?php
    error_reporting(1);

    $target = 'E:\\GitRepository\\coding-webhook'; // 生产环境web目录

    $token = 'photo';
    $wwwUser = 'www';
    $wwwGroup = 'www';

    $json = json_decode(file_get_contents('php://input'), true);

    if (empty($json['token']) || $json['token'] !== $token) {
        exit('error request');
    }

    $repo = $json['repository']['name'];

    $cmd = "cd $target && git pull";

    echo shell_exec($cmd);
~~~

​	确保你的hook文件可以访问：http://51growup.com/coding-webhook/webhook.php，钩子准备完成。

配置Coding的webhook
1.添加用户公钥
复制/root/.ssh/id_rsa.pub内容到个人设置页的SSH公钥里添加即可（https://coding.net/user/account/setting/keys）

2.添加部署公钥
复制/home/www/.ssh/id_rsa.pub的内容并添加到部署公钥:

选择项目 > 设置 > 部署公钥 > 新建 > 粘贴到下面框并确认

3.添加hook
选择项目 > 设置 > WebHook > 新建hook > 粘贴你的coding-webhook/webhook.php所在的网址。比如:http://51growup.com/coding-webhook/webhook.php, 令牌可选，但是建议写上。

稍过几秒刷新页面查看hook状态，显示为绿色勾就OK了。

初始化
1.我们需要先在服务器上clone一次，以后都可以实现自动部署了：
sudo -Hu www git clone https://coding.net/u/itdream6/p/hiyyh/git /data/wwwroot/default/coding-webhook  --depth=1
1
这个时候应该会要求你输入一次Coding的帐号和密码，因为上面我们设置了永久保存用户名和密码，所以之后再执行git就不会要求输入用户名和密码了。

！！注意，这里初始化clone必须要用www用户

2.往Coding.net提交一次代码测试：
在本地clone的仓库执行：

git commit -am "test hook" --allow-empty
git push
1
2

OK，稍过几秒，正常的话你在配置的项目目录里就会有你的项目文件了。






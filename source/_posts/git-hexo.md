---
title: Git与Hexo命令
date: 2017-04-02 11:01:31
header-img:  
tags:
   - git
   - hexo
---

用git如何把项目上传到github
安装就不必多说了，在本地仓库中打开git bash命令窗口，执行如下命令

    $git init

仓库中生成一个隐藏的文件夹.git，用于存放git信息。
1. git配置
（1）本地创建安全连接ssh key
打开git bash命令行窗口，输入下面命令
       $ssh-keygen -t rsa -C "your_email"
“your_email”为你注册github时用的邮箱，之后会要求确认路径和输入密码，这里使用默认的回车就行。成功的话窗口会提醒在用户根目录下生成.ssh文件夹，打开id_rsa.pub，复制里面的key。回到github，进入Account Settings，左边选择SSH and GPG Keys，Add SSH Key,title随便填，粘贴key
![](http://upload-images.jianshu.io/upload_images/4367237-0ad74bf50fbd4dd1.PNG)
（2）验证ssh是否连接成功 

       $ssh -T git@github.com
如果是第一次的会提示是否continue，输入yes就会看到：You’ve successfully authenticated, but GitHub does not provide shell access 。这就表示已成功连上github。（需要注意的是上面的两个命令-C和-T都需要大写）
（3）设置用户名和邮箱
在把本地仓库上传到github之前对git config设置username和email，github会对每次commit进行记录。
       $ git config --global user.name "your name" $ git config --global user.email "your_email@youremail.com"
（4）进入要上传的本地仓库，添加远程连接地址

       $git remote add origin git@github.com:yourname/yourRepo.git

 这里的yourname和yourRepo是你的用户名和github上新建的仓库名。添加完成后可以打开.git文件夹，打开config里面多出了remote “origin”内容，也可以直接在里面添加连接。
2. 提交、上传
（1）将需要上传的文件进行提交
要提交整个项目，不能在项目文件夹上打开git bash，应该在上一次目录下打开，输入命令提价
         $git add "文件名称"$git commit -m "提交注释"
（2）上传到github的master分支

        $git push origin master

 git push 将本地仓库推送到远程服务器。
修改完代码后，使用git status可以查看文件的差别，使用git add 添加要commit的文件，也可以用git add -i来智能添加文件。之后git commit提交本次修改，git push上传到github。
3. .gitignore文件
    .gitignore顾名思义就是告诉git需要忽略的文件，这是一个很重要并且很实用的文件。一般我们写完代码后会执行编译、调试等操作，这期间会产生很多中间文件和可执行文件，这些都不是代码文件，是不需要git来管理的。我们在git status的时候会看到很多这样的文件，如果用git add -A来添加的话会把他们都加进去，而手动一个个添加的话也太麻烦了。这时我们就需要.gitignore了，在文件中将项目中不需要上传的文件后缀写进去即可在上传是忽略。
*Hexo命令学习*
[hexo相关](https://hexo.io/docs/generating.html)

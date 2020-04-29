---
layout: post
title: Gitpages+jekyll搭建个人博客
categories: Tools
description: GitHub+jekyll搭建个人博客
keywords: Gitpages, jekyll
---

本问旨在记录自己手动调试方法

## 1. 软件安装

1. 安装 Ruby。 更多信息请参阅 Ruby 文档中的“[安装 Ruby](https://www.ruby-lang.org/en/documentation/installation/)”。

- Install [Jekyll](https://jekyllrb.com/docs/installation/).
- 安装 Bundler。 更多信息请参阅“[Bundler](https://bundler.io/)”。
  - 安装完 **Ruby** 和 **Jelyll** 后, 创建一个新的Jekyll站点后再开始安装, 目录如下:

![image-20200429162530728](E:\Git\AnnerYang.github.io\images\posts\tools\image-20200429162530728.png)

-  在仓库根目录打开Gemfile文件修改添加如下内容

```shell
source "https://gems.ruby-china.com" #修改为国内源
	gem 'nokogiri'
	gem 'rack', '~> 2.0.1'
	gem 'rspec'
```



## 2. 使用Jekyll创建GitHub Pages网站

### 2.1为站点创建存储仓库

1. 在github上任何页面的右上角，使用下拉菜单选择New repository（新建仓库）。

![带有创建新仓库选项的下拉菜单](E:\Git\AnnerYang.github.io\images\posts\tools\repo-create.png)

2. 使用Owner（所有者）下拉菜单选择你想要拥有仓库的帐户。

![所有者下拉菜单](E:\Git\AnnerYang.github.io\images\posts\tools\create-repository-owner.png)

3. 输入仓库的名称和说明（可选）。 如果您创建的是用户或组织站点，仓库名称必须为`<user>.github.io`或`<organization>.github.io`。 

![创建仓库字段](E:\Git\AnnerYang.github.io\images\posts\tools\create-repository-name-pages.png)

4. 选择存储库可见性。

![Radio buttons to select repository visibility](E:\Git\AnnerYang.github.io\images\posts\tools\create-repository-public-private.png)

### 2.2创建站点

1. 打开 Git Bash。

2. 在本地创建Git目录,并切换到该目录

   ```shell
   $ cd PARENT-FOLDER
   ```

3. 在Git目录初始化仓库信息

   ```shell
   $ git init REPOSITORY-NAME
   > Initialized empty Git repository in /Users/octocat/my-site/.git/
   # Creates a new folder on your computer, initialized as a Git repository
   ```

4. 切换到仓库目录

   ```shell
   $ cd REPOSITORY-NAME
   # Changes the working directory
   ```

5. 创建新的Jekyll site
   ```shell
   $ jekyll VERSION new .
   # Creates a Jekyll site in the current directory
   ```

6. 打开创建的Gemfile，并按照Gemfile注释中的说明使用GitHub Pages

   ```shell
   gem "github-pages","~> 204", group: :jekyll_plugins
   ```

7. 本地启动jekyll服务

   ```shell
   $ bundle exec jekyll serve
   Configuration file: E:/Git/AnnerYang.github.io/_config.yml
               Source: E:/Git/AnnerYang.github.io
          Destination: E:/Git/AnnerYang.github.io/_site
    Incremental build: disabled. Enable with --incremental
         Generating...
          Jekyll Feed: Generating feed for posts
                       done in 0.217 seconds.
    Auto-regeneration: enabled for 'E:/Git/AnnerYang.github.io'
       Server address: http://127.0.0.1:4000/
     Server running... press ctrl-c to stop.
   
   ```

8. 浏览器访问测试

   ```url
   http://localhost:4000
   ```

   ![image-20200429163538394](E:\Git\AnnerYang.github.io\images\posts\tools\image-20200429163538394.png)

9. 将你的GitHub仓库添加为远程存储库，将USER替换为拥有仓库的帐户，并将REPOSITORY替换为仓库的名称

   ```shell
   $ git remote add origin https://github.com/USER/REPOSITORY.git
   ```

10. 将仓库推送到GitHub，将BRANCH替换为您正在处理的分支的名称。

    ```shell
    $ git push -u origin BRANCH
    ```

11. 提交可能会出现提交错误,解决方法如下:

    ```shell
    # 错误信息
    error: src refspec master does not match any.
    error: failed to push some refs to 'git@github.com:hahaha/ftpmanage.git'
    ```

    ```shell
    # 原因：
    # 本地仓库为空
    # 解决方法：使用如下命令 添加文件；
    $ git add add.php addok.php conn.php del.php edit.php editok.php ftpsql.sql index.php
    
    $ git commit -m "init files"
    # 之后在push过程中出现如下错误：
    $ git push -u origin master
    Warning: Permanently added the RSA host key for IP address 'xx.xx.xxx.xxx' to the list of known hosts.
    To git@github.com:hahaha/ftpmanage.git
     ! [rejected]        master -> master (fetch first)
    error: failed to push some refs to 'git@github.com:hahahah/ftpmanage.git'
    hint: Updates were rejected because the remote contains work that you do
    hint: not have locally. This is usually caused by another repository pushing
    hint: to the same ref. You may want to first integrate the remote changes
    hint: (e.g., 'git pull ...') before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.
    # 提示使用 git pull 之后在 push
    # 使用如下命令解决：
    $ git pull --rebase origin master
    warning: no common commits
    remote: Counting objects: 3, done.
    remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
    Unpacking objects: 100% (3/3), done.
    From github.com:hahah/ftpmanage
     * branch            master     -> FETCH_HEAD
     * [new branch]      master     -> origin/master
    First, rewinding head to replay your work on top of it...
    Applying: init files
    ```

    2cd578fb03bd78c5777934e674ae04599b69383e
    
12. ## Jekyll with GitHub Pages - Fix the "GitHub Metadata No GitHub API authentication could be found" Error

    Sep 13, 2016 • • **Jekyll** **GitHub Pages**

    > [YouTube Video](https://www.youtube.com/watch?v=1bQqkyvH5ps)

    Jekyll with GitHub pages is an awesome way to build a developer blog or project site ([find out how](http://knightcodes.com/miscellaneous/2016/04/25/jekyll-on-windows.html)). However, one day after updating my local environment with the latest GitHub Pages gems ([as GitHub tells us to do](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/#keeping-your-site-up-to-date-with-the-github-pages-gem)), I started to get errors about GitHub Metadata and GitHub API authentication anytime I made edits while serving the site locally. WTF?!?! This post will tell you how to fix it.

    ### Overview

    Here is the full error that I was getting in the console:

    ```
    Regenerating: 1 file(s) changed at 2016-09-13 13:10:58    GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
    ...error:
        Error: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed
        Error: Run jekyll build --trace for more information.
    ```

    Initially the site would generate and be served successfully, but if I made any edits I would get the error in my terminal and my edits wouldn’t be served. This means every time I made a change and wanted to see what it looked like, I would have to terminate the Jekyll process and have it regenerate the site all over again. So annoying!

    ### Fix It!

    Alas, after much searching I found a fix. Just follow these steps:

    1. Create a personal access token in GitHub. This takes just a second, see [GitHub’s documentation](https://help.github.com/articles/creating-an-access-token-for-command-line-use/). When picking the scope(s) that you want to grant to the token, just select the `repo` checkbox.
    2. Add a new system environment variable on your machine named `JEKYLL_GITHUB_TOKEN` and set the value equal to the personal access token you generated.
    3. [Go here](https://curl.haxx.se/ca/cacert.pem), copy all the text in the page, and save it as a file named `cacert.pem` somewhere on your local machine.
    4. Add a new system environment variable on your machine named `SSL_CERT_FILE` and set the value equal to the full file path of where you saved the `cacert.pem` file.
    5. If you are on **Windows** then you must **restart your machine**.

    You are now on your merry way to being able to make edits while the Jekyll site is being served.
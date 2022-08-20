# 1 Git基础

## 1.1获取文件仓库

```shell
git init
```

该命令会创建一个名为.git的子目录

```shell
git clone [url] 
```

该命令会克隆该仓库服务器上所有数据，默认配置下的远程Git仓库中每一个文件的每一个版本都将会被拉取下来。

## 1.2 记录更新

![image-20220817223541727](E:\Computer Science\个人笔记\工具\精通git.asset\image-20220817223541727-16607470099582.png)

文件状态：

- 已跟踪
  - 没有更改
  - 已更改
  - 放入暂存区
- 未跟踪

```shell
git status #查看文件状态
```

```shell
git add [file] #将文件进行跟踪
```

```shell
git mv file_from file_to #对文件进行重命名，如果直接重命名，git不会知道这是重命名的操作
```

```shell
git log #查看历史
git log -p -2 #最近两次的提交
```

补交漏掉的文件

```shell
git commit -m "initial commit"
git add forgetten_file
git commit --amend	#最终只会有一个提交，第二次提交将会代替第一次提交的结果
```

取消暂存或者修改的文件

```shell
git reset HEAD CONTRIBUTING.md #取消对CONTRIBUTING文件的暂存，
git checkout --CONTRIBUTING.md #取消对CONTRIBUTING文件的修改
```

## 1.3 远程仓库的使用

添加远程仓库

```shell
git remote add <shortname> <url>
```

```shell
git fetch [remote-name] #将数据拉取到本地仓库，但不会自动合并或修改当前工作文档
git pull [remote-name] #会自动抓取然后合并远程分支到当前分支
```

打标签，标签分为轻量标签和附注标签

附注标签是存储在git数据库中的一个完整对象，可以被校验，其中包含打标签的名字、电子邮件地址、日期时间

```shell
git tag -a v1.4 -m 'my version 1.4' #-a表示附注标签，-m表示指定了一条将会存储在标签中的信息
```

轻量标签本质上是将提交校验和存储到一个文件中，没有保存任何其他信息

```shell
git tag v1.4-lw
```

对过去提交打标签

```shell
git tag -a v1.2 9fceb02	#需要在命令的末尾指定提交的校验和（或部分校验和）
```

git 必须显式的推送标签

```shell
git push origin [tagname]
```


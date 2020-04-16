---
title: git相关
tags:
---

#### 常用命令

1. 查看本地仓库对应的远程地址：

$ git remote -v

2. **切换为ssh通道**

$ git remote set-url origin git的ssh地址

3. 删除本地缓存

#### 多账号问题

解决公司gitlab账号和个人github账号，用户名串用问题

1. 哪个账号对应的项目比较多，将其用户名、邮箱设置为global的

   ```java
   git config --global user.name "Username1"
   git config --global user.email "Email1"
   ```

2. 在需要使用另一个用户名、邮箱的地方，设置项目的Local信息，覆盖global信息

   ```java
   git config --local user.name "Username2"
   git config --local user.email "Email2"
   ```

#### 拉取代码相关

1. 拉取(clone)指定分支代码

   git clone **-b BRANCH_NAME** git@xxxx

2. 检出(checkout)指定分支代码

   git checkout **-b** BRANCH_NAME **origin**/BRANCH_NAME

<<<<<<< HEAD
#### Rebase



#### Reset

**reset** 这个指令虽然可以用来撤销 **commit** ，但它的实质行为并不是撤销，而是移动 **HEAD** ，并且「捎带」上 **HEAD** 所指向的 **branch**（如果有的话）。也就是说，**reset** 这个指令的行为其实和它的字面意思 "**reset**"（重置）十分相符：它是用来重置 **HEAD** 以及它所指向的 **branch** 的位置的。有三种模式：mixed(默认)/soft/hard

+ mixed
+ soft
+ hard



#### Cherry-Pick





=======


#### 问题记录

1. 拉取代码后，权限不同，修改了权限，显示所有文件都被modified了，使用git diff，显示:

   ```java
   old mode 100755 
   new mode 100644
   ```

   解决办法：解决方法：

   ```java
   git config --global core.filemode false
   git config core.filemode false
   ```

2. 如何回滚已经提交到远程的commit

   ```java
   回退命令：
   $ git reset --hard HEAD^         回退到上个版本
   $ git reset --hard HEAD~3        回退到前3次提交之前，以此类推，回退到n次提交之前
   $ git reset --hard commit_id     退到/进到 指定commit的sha码
   
   强推到远程：
   $ git push origin HEAD --force
   ```

3. Could Not Merge wanly_67886: refusing to merge unrelated histories

   

4. 


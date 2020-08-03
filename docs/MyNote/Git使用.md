# Git使用基础

> **笔记来自[菜鸟教程](<https://www.runoob.com/git/git-basic-operations.html>)**

## 一、Git基本操作

1. 创建Git仓库：

   ```shell
   git init
   ```

2. 拷贝一个仓库到本地

   ```shell
   git clone <repo> # 基本操作
   git clone <repo> <directory>  # 克隆到指定的目录
   ```

3. 查看项目的当前状态

   ```shell
   git status # 详细输出
   git status -s # 获得简短的结果输出
   ```

4. 将文件添加到缓存

   ```shell
   git add README hello.php # 指定文件名
   git add . # 添加当前项目的所有文件
   ```

5. git diff 命令显示已写入缓存与已修改但尚未写入缓存的改动的区别。git diff 有两个主要的应用场景。

   - 尚未缓存的改动：**git diff**
   - 查看已缓存的改动： **git diff --cached**
   - 查看已缓存的与未缓存的所有改动：**git diff HEAD**
   - 显示摘要而非整个 diff：**git diff --stat**

6. 将缓存区内容添加到仓库中

   ```shell
   git commit -m '提交备注'
   git commit -a # 直接跳过 git add 提交缓存的流程
   ```

7. 取消已缓存的内容

   ```shell
   git reset HEAD <file>
   ```

8. 如果只是简单地从工作目录中手工删除文件，运行 **git status** 时就会在 **Changes not staged for commit** 的提示。

   要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除，然后提交。可以用以下命令完成此项工作:

   ```shell
   git rm <file>
   ```

   如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 **-f**

   ```shell
   git rm -f <file>
   ```

9. 移动或重命名一个文件、目录、软连接。

   ```shell
   git mv
   ```

## 二、Git分支管理

1. 创建分支命令：

   ```shell
   git branch (branchname)
   ```

2. 切换分支命令:

   ```shell
   git checkout (branchname)
   ```

3. 合并分支命令:

   ```shell
   git merge
   ```

4. 列出分支基本命令：

   ```shell
   git branch
   ```

5. 手动创建一个分支:

   ```shell
   git branch (branchname)
   ```

   也可以使用 

   ```shell
   git checkout -b (branchname) 
   ```

   命令来创建新分支并立即切换到该分支下，从而在该分支中操作。

6. 删除分支命令：

   ```shell
   git branch -d (branchname)
   ```

## 三、Git 查看提交历史

1. 在使用 Git 提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，我们可以使用 git log 命令查看。

   ```shell
   git log
   git log --oneline # 查看历史记录的简洁的版本
   git log --reverse --oneline # 用 --reverse 参数来逆向显示所有日志
   git log --author=Linus --oneline # 查找指定用户的提交日志
   ```

   我们还可以用 --graph 选项，查看历史中什么时候出现了分支、合并。

   如果你要指定日期，可以执行几个选项：--since 和 --before，但是你也可以用 --until 和 --after。

   例如，如果我要看 Git 项目中三周前且在四月十八日之后的所有提交，我可以执行这个（我还用了 --no-merges 选项以隐藏合并提交）

   ```shell
   git log --oneline --before={3.weeks.ago} --after={2010-04-18} --no-merges
   ```

## 四、Git标签

1. 如果你达到一个重要的阶段，并希望永远记住那个特别的提交快照，你可以使用 git tag 给它打上标签。比如说，我们想为我们的 runoob 项目发布一个"1.0"版本。 我们可以用 git tag -a v1.0 命令给最新一次提交打上（HEAD）"v1.0"的标签。-a 选项意为"创建一个带注解的标签"。 不用 -a 选项也可以执行的，但它不会记录这标签是啥时候打的，谁打的，也不会让你添加个标签的注解。 我推荐一直创建带注解的标签。

   ```shell
   git tag -a v1.0 
   ```

   当你执行 git tag -a 命令时，Git 会打开你的编辑器，让你写一句标签注解，就像你给提交写注解一样。现在，注意当我们执行 git log --decorate 时，我们可以看到我们的标签了。

2. 查看所有标签

   ```shell
   git tag
   ```

## 五、Git 远程仓库(Github)

1. 添加远程库

   要添加一个新的远程仓库，可以指定一个简单的名字，以便将来引用,命令格式如下：

   ```shell
   git remote add [shortname] [url]
   ```

   由于你的本地 Git 仓库和 GitHub 仓库之间的传输是通过SSH加密的，所以我们需要配置验证信息：

   使用以下命令生成 SSH Key：

   ```shell
   $ ssh-keygen -t rsa -C "youremail@example.com"
   ```

   后面的 your_email@youremail.com 改为你在 Github 上注册的邮箱，之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。成功的话会在 ~/ 下生成 .ssh 文件夹，进去，打开 **id_rsa.pub**，复制里面的 **key**

   ```shell
   # 提交到 Github
   $ git remote add origin git@github.com:tianqixin/runoob-git-test.git
   $ git push -u origin master
   ```

2. 提取远程仓库

   a、从远程仓库下载新分支与数据：

   ```shell
   git fetch
   ```

   该命令执行完后需要执行git merge 远程分支到你所在的分支。

   b、从远端仓库提取数据并尝试合并到当前分支：

   ```shell
   git merge
   ```

   该命令就是在执行 git fetch 之后紧接着执行 git merge 远程分支到你所在的任意分支。

   假设你配置好了一个远程仓库，并且你想要提取更新的数据，你可以首先执行 **git fetch [alias]** 告诉 Git 去获取它有你没有的数据，然后你可以执行 **git merge [alias]/[branch]** 以将服务器上的任何更新（假设有人这时候推送到服务器了）合并到你的当前分支。

3. 推送你的新分支与数据到某个远端仓库命令:

   ```shell
   git push [alias] [branch]
   ```

   以上命令将你的 [branch] 分支推送成为 [alias] 远程仓库上的 [branch] 分支

4. 删除远程仓库你可以使用命令：

   ```shell
   git remote rm [别名]
   ```

5. git config

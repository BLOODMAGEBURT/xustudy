### git 合并分支的流程

> 目前的情况：
>
> 处在master分支中，需要合并issue-234分支到master分支

#### 步骤

1. 先处理master分支的代码

   a. git pull

   b. git add .

   c. git commit -m " new "

   d. git push

2. 合并分支

   git merge issue-234

3. 提交更新

   1. git pull
   2. git add . 
   3. git commit -m " merge issue"
   4. git push

如果合并中出现冲突，就解决冲突，然后再次commit -am 
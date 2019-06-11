### Git 操作命令

- 查看分支

  - 查看本地分支

    `git branch`

  - 查看所有分支

    `git branch -a`

- 全局配置

  - 查看用户名、邮箱

    `git config user.name`

    `git config user.email`

  - 修改用户名、邮箱

    `git config user.name "your name"`

    `git config user.email "your email"`

- 合并分支流程

  > 1. 开发某个网站。
  > 2. 为实现某个新的需求，创建一个分支。
  > 3. 在这个分支上开展工作。
>
  > **分支相关的操作，要确保分支是干净的：**
>
  > a. 切换分支之前先提交本分支(commit)
  >
  > b. 主分支合并功能分支前，先提交主分支

  1. 分支的新建与切换

     ```shell
     git checkout -b newFunction
     ```
  
  2. 提交在此分支的修改
  
     ```shell
      git commit -am 'added a new footer [issue 53]'
        
      此命令相当于以下两步
     ```

    git add .
      git commit -m 'added a new footer [issue 53]'
      
     ```

  3. 切换到主分支

     ```shell
     git checkout master
     ```

  4. 合并新功能的分支

     ```shell
     git merge newFunction
     ```

  5. 合并完成后，将新功能分支删除

     ```shell
      git branch -d newFunction
     ```

  > 有时候合并操作并不会如此顺利。如果在不同的分支中都修改了同一个文件的同一部分，Git 就无法干净地把两者合到一起（译注：逻辑上说，这种问题只能由人来裁决。）。如果你在解决问题 #53 的过程中修改了 `hotfix` 中修改的部分，将得到类似下面的结果：

  ​		

  ​		

```shell
  $ git merge iss53
Auto-merging index.html
  CONFLICT (content): Merge conflict in index.html
  Automatic merge failed; fix conflicts and then commit the result.
```

  1. Git 作了合并，但没有提交，它会停下来等你解决冲突。要看看哪些文件在合并时发生冲突，可以用 `git status` 查阅：
  
     ```shell
     $ git status
     On branch master
     You have unmerged paths.
       (fix conflicts and run "git commit")
     
     Unmerged paths:
     (use "git add <file>..." to mark resolution)
     
           both modified:      index.html
     
     no changes added to commit (use "git add" and/or "git commit -a")
     ```
  
  2. 任何包含未解决冲突的文件都会以未合并（unmerged）的状态列出。Git 会在有冲突的文件里加入标准的冲突解决标记，可以通过它们来手工定位并解决这些冲突。可以看到此文件包含类似下面这样的部分：
  
     ```html
     <<<<<<< HEAD
     <div id="footer">contact : email.support@github.com</div>
     =======
     <div id="footer">
       please contact us at support@github.com
      </div>
     >>>>>>> iss53
      
     可以看到 ======= 隔开的上半部分，是 HEAD（即 master 分支，在运行 merge 命令时所切换到的分支）中的内容，下半部分是在 iss53 分支中的内容。
     
     ```
  
  3. 解决冲突的办法无非是二者选其一或者由你亲自整合到一起。比如你可以通过把这段内容替换为下面这样来解决：
  
     ```html
     <div id="footer">
     please contact us at email.support@github.com
      </div>
     
     ```

   这个解决方案各采纳了两个分支中的一部分内容，
     而且我还删除了
     <<<<<<<，======= 和 >>>>>>> 这些行。
     ```

  4. 在解决了所有文件里的所有冲突后，运行 `git add` 将把它们标记为已解决状态（译注：实际上就是来一次快照保存到暂存区域。）。因为一旦暂存，就表示冲突已经解决。

     ```shell
     git add .
     ```
  
  5. 再运行一次 `git status` 来确认所有冲突都已解决：
  
     ```shell
     $ git status
      On branch master
     Changes to be committed:
     (use "git reset HEAD <file>..." to unstage)
     
             modified:   index.html
     ```
  
  6. 如果觉得满意了，并且确认所有冲突都已解决，也就是进入了暂存区，就可以用 `git commit` 来完成这次合并提交。提交的记录差不多是这样：
  
     ```shell
     $ git commit -m "conflict fixed"
     
     Merge branch 'iss53'
     
     Conflicts:
       index.html
     #
     # It looks like you may be committing a merge.
     # If this is not correct, please remove the file
      #       .git/MERGE_HEAD
      # and try again.
      #  如果想给将来看这次合并的人一些方便，可以修改该信息，提供更多合并细节。比如你都作了哪些改动，以及这么做的原因。有时候裁决冲突的理由并不直接或明显，有必要略加注解。
     
     ```



  7. 提交到远程仓库，供同伴使用
  
     ```shell
     git push
     ```

- 重命名文件/文件夹

  ​	

  ```shell
  git mv oldName newName
  git commit -am 'old to new commit'
  git push
  ```

- 远程分支

  - 新建远程分支

    ```shell
    git push origin <branchName>
    ```

  - 删除远程分支

    ```shell
    git push origin -d <branchName>
    ```

    

  > 将本地的文件，文件夹上传到`github`
  
  - 初始化本地仓库
  
    ```shell
    git init
    ```
  
  - 提交文件，文件夹
  
    ```shell
    git add.
    git commit -m 'first blood'
    ```
  
  - 关联`github`仓库
  
    - 在github中新建一个repository，复制仓库地址
  
    - #### **添加远程地址**
  
      ```shell
      git remote add origin git@github.com:BLOODMAGEBURT/xustudy.git
      
      
      #注意：
      
      如果出现错误：fatal: remote origin already exists，则执行以下语句：
      
          $ git remote rm origin
      再重新执行：
      
         $ git remote add origin git@github.com:BLOODMAGEBURT/xustudy.git
      --------------------- 
      ```
  
    - 将缓存区推送到远程仓库
  
      ```shell
      git push origin master
      
      
      # 如果出现错误failed to push som refs to…….，则执行以下语句，先把远程服务器 
      # github上面的文件拉先来，再push 上去。
         $ git pull origin master
      ```
  
    - 刷新`github`，即可看到上传的文件夹。

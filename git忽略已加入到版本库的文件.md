### git忽略已加入到版本库的文件

- #### 遇到的问题

  项目中，我们会用到 `'.gitignore'` 来忽略一些文件，不记录这些文件的版本控制。
  然而，经常发现，已经添加到了 '`.gitignore'` 的文件/目录，每次的修改等扔会记录版本。
  产生这种原因，一般都是由于，在初始项目时，已经使用 git add 将该文件，加入到了版本库

- 解决方案

  1. `git rm -r --cached target-dir`

  2. git commit -m  "从版本库移除`target-dir`目录"

  3. git push

  4. 此时我们发现，仓库里的target-dir目录已经不见了

     解释：

     git rm 的选项：
     -r    	        // 递归删除目录
     --cached 	// 我们本次核心使用，不记录到版本库

- 进阶

  git rm 和 git rm -r --cached 区别：

  ------

  当我们需要删除暂存区或分支上的文件，同时工作区 '不需要' 这个文件，可以使用 'git rm'
  git rm file
  git commit -m 'delete file'

  git push

  ------

  当我们需要删除暂存区或分支上的文件，但是本地 '需要' 这个文件，只是 '不希望加入版本控制'，可以使用 'git rm -r --cached'	
  git rm -r --cached target-dir	
  git commit -m 'delete remote file'
  git push
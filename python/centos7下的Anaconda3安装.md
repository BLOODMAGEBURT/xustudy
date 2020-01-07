### centos7下的Anaconda3安装

1. 确认是否已经安装bzip2

   ​	直接在终端输入bzip2并回车，如果没有提示 `command not found`，说明已经安装

   ​	如果没有安装，执行以下命令

   ​	`sudo yum install -y bzip2`

2. 然后去Anaconda 的 [清华镜像源 ](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)找到你想要的版本，然后拷贝下载地址

   ​	例如我要的是 <https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.0.1-Linux-x86_64.sh>

   ​	然后去linux 下执行命令下载

   ​	wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.0.1-Linux-x86_64.sh

3. 下载完成后，执行

   ​	bash Anaconda3-5.0.1-Linux-x86_64.sh

   ​	后面一直选yes就行了

4. 加入环境变量,**（只对当前登陆用户生效，永久生效）**

   ​	执行 vim ~/.bash_profile 修改文件中 PATH 一行，将 /root/anaconda3/bin 

   ​	加入到PATH=$PATH:$HOME/bin 一行之后（注意以冒号分隔），

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200107180349.png)

   ​	保存文件并退出，

   ​	执行 source ~/.bash_profile 使其生效，这种方法只对当前登陆用户生效。

5. 最后执行以下命令，激活环境

   ​	source ~/.bashrc

   ​	安装完成，重启终端，输入python3

   ![1555037474632](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555037474632.png)

   ​	显示已经使用Anaconda

------

### Anaconda的操作

-  查看所有的环境

  conda env list

-  新建一个环境

  conda create -n env_name  list of packages

  其中 `-n` 代表 name，`env_name` 是需要创建的环境名称，`list of packages` 则是列出在新环境中需要安装的工具包。

    

  例如 ： 

    我还需要创建一个 Python 2 的环境来运行旧版本的 Python 代码，还安装 pandas 包，于是我们运行以下命令来创建：

  

  | 1    | conda create -n py2 python=2.7 pandas |
  | ---- | ------------------------------------- |
  |      |                                       |

    细心的你一定会发现，py2 环境中不仅安装了 pandas，还安装了 numpy 等一系列 packages，

    这就是使用 conda 的方便之处，它会自动为你安装相应的依赖包，而不需要你一个个手动安装。

- 进入环境，例如进入名为 env_name 的环境

  source activate env_name

- 退出环境

  source deactivate

- 删除环境，例如删除名为 env_name 的环境

  conda env remove -n env_name

### 进阶操作：

当分享代码的时候，同时也需要将运行环境分享给大家，执行如下命令可以将当前环境下的 package 信息存入名为 environment 的 YAML 文件中。

| 1    | conda env export > environment.yaml |
| ---- | ----------------------------------- |
|      |                                     |

同样，当执行他人的代码时，也需要配置相应的环境。这时你可以用对方分享的 YAML 文件来创建一摸一样的运行环境。



| 1    | conda env create -f environment.yaml |
| ---- | ------------------------------------ |
|      |                                      |


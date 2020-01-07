### Venv 操作指南

- 首先安装virtualenv

  `pip install virtualenv`

- 创建虚拟环境并制定python版本

  `virtualenv venv --python=python2**.7**

  virtualenv venv --python=python3.5

  virtualenv venv --python=python3.6`

- 激活环境

  - mac 环境

    `source venv/bin/activate`

  - windows环境

    `venv\Scripts\activate.bat`

- 退出虚拟环境

  `deactivate`

- 删除虚拟环境

  - mac 环境

    `rm -r venv`

  - windows 环境

    rd /s /q venv

    *因为rd只能删除空的文件夹，而如果其中有子文件或子文件夹的时候就会停下来，*
    *这时我们加上 /s 就可以直接删除，但是删除过程中会提示你是否确定删除，对于懒癌患者我们有添加了 /q，即quiet，安静模式；所以使用以上命令会完整删除你选中的整个文件夹。*

  


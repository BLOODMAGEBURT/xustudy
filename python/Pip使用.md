### Pip使用

- 自动补全

  ```shell
  pip completion --bash >> ~/.bashrc
  ```
  
- 冻结与修复

  ```shell
  # 冻结
  pip freeze > requirements.txt
  
  # 安装 requirements.txt中的扩展
  pip install -r requirements.txt
  ```

- 使用加速镜像地址

  ```shell
  # 使用清华的镜像源
  pip install flask redis -i https://pypi.tuna.tsinghua.edu.cn/simple
  ```

  
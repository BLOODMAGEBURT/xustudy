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

  
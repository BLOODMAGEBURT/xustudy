### flask-migrate  修改 sqlite的表头

- 遇到的问题

  我将Notification的其中一个列名name 修改为 type

  ![1550569934871](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550569934871.png)

  然后，执行

  `flask db migrate -m "notification -name to type"`

    木有出现问题，好消息，那就继续

    然后，又执行

  `flask db upgrade`

    悲剧了，报错了

    `sqlite3.OperationalError: near "DROP": syntax error`

- 解决方式

  1.修改migrations/evn.py中的方法， 添加 render_as_batch=True

  ![1550570971545](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550570971545.png)

  2. 将versions文件夹下刚才新生成的脚本删除

     ![1550571093913](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550571093913.png)

  3. 重新运行 flask db migrate -m "notification -- name to type"

  4. 然后 flask db upgrade 就正常运行了
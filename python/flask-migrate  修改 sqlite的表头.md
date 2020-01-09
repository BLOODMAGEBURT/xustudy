### flask-migrate  修改 sqlite的表头

- 遇到的问题

  我将Notification的其中一个列名name 修改为 type

  ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109155010.png)

  然后，执行

  `flask db migrate -m "notification -name to type"`

    木有出现问题，好消息，那就继续

    然后，又执行

  `flask db upgrade`

    悲剧了，报错了

    `sqlite3.OperationalError: near "DROP": syntax error`

- 解决方式

  1.修改migrations/evn.py中的方法， 添加 render_as_batch=True

  ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109155036.png)

  2. 将versions文件夹下刚才新生成的脚本删除

     ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109155058.png)

  3. 重新运行 flask db migrate -m "notification -- name to type"

  4. 然后 flask db upgrade 就正常运行了
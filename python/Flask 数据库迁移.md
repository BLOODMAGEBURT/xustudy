### Flask 数据库迁移

- 准备工作

  - 安装扩展

    - ```python
      pip install flask-sqlalchemy
      pip install flask-migrate
      ```

  - ```python
    from flask_sqlalchemy import SQLAlchemy
    from flask_migrate import Migrate
    
    app = Flask(__name__)  # type:Flask
    app.config.from_object(Config)
    db = SQLAlchemy(app)  # type:SQLAlchemy
    migrate = Migrate(app, db)
    ```

  - Config.py

    ```python
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or 'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    ```

- ## Database Models

  ```python
  from app import db
  
  class User(db.Model):
      id = db.Column(db.Integer, primary_key=True)
      username = db.Column(db.String(64), index=True, unique=True)
      email = db.Column(db.String(120), index=True, unique=True)
      password_hash = db.Column(db.String(128))
  
      def __repr__(self):
          return '<User {}>'.format(self.username) 
  ```

- ## Creating The Migration Repository

  `flask db init`

- ## The First Database Migration

  `flask db migrate -m "users table"`

  `flask db upgrade`

  

- ## Database Upgrade and Downgrade Workflow

  > With database migration support, after you modify the models in your application you generate a new migration script (`flask db migrate`), you probably review it to make sure the automatic generation did the right thing, and then apply the changes to your development database (`flask db upgrade`). You will add the migration script to source control and commit it.
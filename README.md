# Flask开发video网站

标签（空格分隔）： python

---

### 环境搭建

### Virtualenv环境搭建
```
pip3 install virtualenv

virtualenv venv（创建venv目录）
#最终会在/Users/wuyinlei/venv

ls venv/
#会显示 bin  lib  include三个目录和pip-selfcheck.json文件

venv/bin/activate

pip3 freeze(检测有什么import包 正常第一次什么都不反应)
#安装flask
pip3 install flask
#这个时候在执行pip3 freeze会显示如下安装包
click==6.7
Flask==0.12.2
itsdangerous==0.24
Jinja2==2.10
MarkupSafe==1.0
Werkzeug==0.14.1

#退出虚拟化环境
deactivate

```

### 项目创建
pycharm-->File-->New Project-->Python(3.x版本)

#### 项目测试

* 新建app.py
```
# coding=utf8
from flask import Flask

app = Flask(__name__)


@app.route("/")
def index():
    return "<h1 style='color:red'>Hello World!</h1>"


if __name__ == '__main__':
    app.run()

```
点击运行可以看到控制台会有一下代码:
```
/Library/Frameworks/Python.framework/Versions/3.5/bin/python3.5 /Users/wuyinlei/PycharmProjects/movie_project/app.py
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

```
这个时候点击`http://127.0.0.1:5000/`会在网页中显示红色的`Hello World!` 代表项目创建成功

#### 预计项目结构
manage.py               入口启动脚本
app                     项目app
    __init__.py         初始化文件
    models.py           数据模型文件
    static              静态目录
    home/admin          前后台模块
        __init__.py     初始化脚本
        views.py        视图处理文件
        forms.py        表单处理文件
    templates           模板目录
        home/admin      前后台模板

#### 蓝图构建项目目录
* 1、什么是蓝图?
    * 一个应用中或跨应用制作应用组件和支持通用的模式
* 2、蓝图的作用
    * 将不同的功能模块化
    * 构建大型应用
    * 优化项目结构
    * 增强可读性、易于维护


##### 1、定义蓝图(app/admin/__init__.py)
```
from flask import Blueprint
admin = Blueprint("admin",__name__)
import views
```
##### 2、注册蓝图(app/__init__.py)
```
from admin import admin as admin_blueprint
app.register_blueprint(admin_blueprint,url_prefix="/admin")
```
##### 3、调用蓝图(app/admin/views.py)
```
from .import admin
@admin.route("/")
```

#### 会员及会员登录日志数据模型设计
##### 1、安装数据库连接依赖包
```
pip3 install flask-sqlalchemy
```
##### 2、定义mysql数据库连接
```
from flask_sqlalchemy import SQLALchemy
from flask import Flask

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI']="mysql://root:root@localhost/movie"
app.config['SQLALCHEMY_TRACK_MODIFICATIONS']=Ture
db = SQLALchemy(app)
```

##### 会员数据模型
```
# 会员
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)  # 编号
    name = db.Column(db.String(100), unique=True)  # 昵称
    pwd = db.Column(db.String(100))  # 密码
    email = db.Column(db.String(100), unique=True)  # 邮箱
    phone = db.Column(db.String(11), unique=True)  # 手机号
    info = db.Column(db.Text)  # 简介
    face = db.Column(db.String(255), unique=True)  # 头像
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 注册时间
    uuid = db.Column(db.String(255), unique=True)  # 唯一标识符
    userlogs = db.relationship('UserLog', backref='user')  # 会员日志外键关系关联
    comments = db.relationship('Comment', backref='user')  # 评论外键关系关联
    moviecols = db.relationship('Moviecol', backref='user')  # 电影收藏外键关系关联

    def __repr__(self):
        return "<User %r>" % self.name

```
##### 会员登录日志模型
```
# 会员登录日志
class UserLog(db.Model):
    __tablename__ = 'userlog'
    id = db.Column(db.Integer, primary_key=True)  # 编号
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 所属会员编号
    ip = db.Column(db.String(100))  # 最近登录ip地址
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 最近登录时间

    def __repr__(self):
        return "<UserLog %r>" % self.id
```

##### 标签数据模型
```
# 标签
class Tag(db.Model):
    __tablename__ = 'tag'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), unique=True)
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 最近登录时间
    movie = db.relationship("Movie", backref='tag')  # 电影外键关系关联

    def __repr__(self):
        return "<Tag %r>" % self.name
```

##### 电影模型
```
# 电影
class Movie(db.Model):
    __tablename__ = 'movie'
    id = db.Column(db.Integer, primary_key=True)  # 标号
    title = db.Column(db.String(255), unique=True)  # 标题
    url = db.Column(db.String(255), unique=True)  # 地址
    info = db.Column(db.Text)  # 简介
    logo = db.Column(db.String(255), unique=True)  # logo
    star = db.Column(db.SmallInteger)  # 星级
    playnum = db.Column(db.BigInteger)  # 播放量
    commentnum = db.Column(db.BigInteger)  # 评论量
    tag_id = db.Column(db.Integer, db.ForeignKey('tag.id'))  # 所属标签
    area = db.Column(db.String(255))  # 地址
    release_time = db.Column(db.Date)  # 上映时间
    length = db.Column(db.String(100))  # 长度
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间
    comments = db.relationship("Comment", backref='movie')  # 评论外键关系关联
    moviecols = db.relationship('Moviecol', backref='movie')  # 电影收藏外键关系关联

    def __repr__(self):
        return "<Movie %r>" % self.title
```
##### 上映预告模型
```
# 上映预告
class Preview(db.Model):
    __tablename__ = 'preview'
    id = db.Column(db.Integer, primary_key=True)  # 标号
    title = db.Column(db.String(255), unique=True)  # 标题
    logo = db.Column(db.String(255), unique=True)  # logo
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间

    def __repr__(self):
        return "<Preview % r>" % self.title
```

##### 评论数据模型
```
# 评论
class Comment(db.Model):
    __tablename__ = "comment"
    id = db.Column(db.Integer, primary_key=True)  # 标号
    content = db.Column(db.Text)  # 内容
    movie_id = db.Column(db.Integer, db.ForeignKey('movie.id'))  # 所属电影
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 所属用户
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间

    def __repr__(self):
        return "<Comment % r>" % self.id
```

##### 电影收藏模型
```

# 电影收藏
class Moviecol(db.Model):
    __tablename__ = 'moviecol'
    id = db.Column(db.Integer, primary_key=True)  # 标号
    movie_id = db.Column(db.Integer, db.ForeignKey('movie.id'))  # 所属电影
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 所属用户
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间

    def __repr__(self):
        return "<Moviecol % r>" % self.id
```

##### 权限模型
```
# 权限模型
class Auth(db.Model):
    __tablename__ = 'auth'
    id = db.Column(db.Integer, primary_key=True)  # 标号
    name = db.Column(db.String(100), unique=True)  # 权限名称
    url = db.Column(db.String(255), unique=True)  # 权限地址
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间

    def __repr__(self):
        return "<Auth % r>" % self.name

```

##### 角色模型
```
# 角色模型
class Role(db.Model):
    __tablename__ = 'role'
    id = db.Column(db.Integer, primary_key=True)  # 标号
    name = db.Column(db.String(100), unique=True)  # 权限名称
    auths = db.Column(db.String(600))  # 角色权限列表
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间

    def __repr__(self):
        return "<Role % r>" % self.name
```

##### 管理员数据模型
```

# 管理员数据模型
class Admin(db.Model):
    __tablename__ = 'admin'
    id = db.Column(db.Integer, primary_key=True)  # 编号
    name = db.Column(db.String(100), unique=True)  # 管理员账号
    pwd = db.Column(db.String(100))  # 密码
    is_super = db.Column(db.SmallInteger)  # 是否是超级管理员 0 -> 超级管理员
    role_id = db.Colum(db.Integer, db.ForeignKey('role.id'))  # 所属角色
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 添加时间
    adminlogs = db.relationship('Adminlog', backref='admin')  # 管理员登录日志外键关系关联
    oplogs = db.relationship('Oplog', backref='oplog')  # 管理员操作日志外键关系关联

    def __repr__(self):
        return "<Admin % r>" % self.name

```
##### 登录日志模型
```
# 登录日志模型
class AdminLog(db.Model):
    __tablename__ = 'adminlog'
    id = db.Column(db.Integer, primary_key=True)  # 编号
    admin_id = db.Column(db.Integer, db.ForeignKey('admin.id'))  # 所属会员编号
    ip = db.Column(db.String(100))  # 最近登录ip地址
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 最近登录时间

    def __repr__(self):
        return "<AdminLog %r>" % self.id
```
##### 操作日志数据模型
```
# 操作日志数据模型
class OpLog(db.Model):
    __tablename__ = 'oplog'
    id = db.Column(db.Integer, primary_key=True)  # 编号
    admin_id = db.Column(db.Integer, db.ForeignKey('admin.id'))  # 所属会员编号
    ip = db.Column(db.String(100))  # 最近登录ip地址
    reason = db.Column(db.String(600))  # 操作原因
    addtime = db.Column(db.DateTime, index=True, default=datetime.now)  # 最近登录时间

    def __repr__(self):
        return "<OpLog %r>" % self.id
```
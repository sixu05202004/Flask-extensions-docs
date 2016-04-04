.. _quickstart:

快速入门
==========

.. currentmodule:: flask.ext.sqlalchemy

Flask-SQLAlchemy 使用起来非常有趣，对于基本应用十分容易使用，并且对于大型项目易于扩展。有关完整的指南，请参阅 :class:`SQLAlchemy` 的 API 文档。

一个最小应用
---------------

常见情况下对于只有一个 Flask 应用，所有您需要做的事情就是创建 Flask 应用，选择加载配置接着创建 :class:`SQLAlchemy` 对象时候把 Flask 应用传递给它作为参数。

一旦创建，这个对象就包含 :mod:`sqlalchemy` 和 :mod:`sqlalchemy.orm` 中的所有函数和助手。此外它还提供一个名为 `Model` 的类，用于作为声明模型时的 delarative 基类::

    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:////tmp/test.db'
    db = SQLAlchemy(app)


    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)

        def __init__(self, username, email):
            self.username = username
            self.email = email

        def __repr__(self):
            return '<User %r>' % self.username

为了创建初始数据库，只需要从交互式 Python shell 中导入 `db` 对象并且调用 :meth:`SQLAlchemy.create_all` 方法来创建表和数据库:

>>> from yourapplication import db
>>> db.create_all()

Boom, 您的数据库已经生成。现在来创建一些用户:

>>> from yourapplication import User
>>> admin = User('admin', 'admin@example.com')
>>> guest = User('guest', 'guest@example.com')

但是它们还没有真正地写入到数据库中，因此让我们来确保它们已经写入到数据库中:

>>> db.session.add(admin)
>>> db.session.add(guest)
>>> db.session.commit()

访问数据库中的数据也是十分简单的:

>>> users = User.query.all()
[<User u'admin'>, <User u'guest'>]
>>> admin = User.query.filter_by(username='admin').first()
<User u'admin'>

简单的关系
--------------------

SQLAlchemy 连接到关系型数据库，关系型数据最擅长的东西就是关系。因此，我们将创建一个使用两张相互关联的表的应用作为例子::


    from datetime import datetime


    class Post(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        title = db.Column(db.String(80))
        body = db.Column(db.Text)
        pub_date = db.Column(db.DateTime)

        category_id = db.Column(db.Integer, db.ForeignKey('category.id'))
        category = db.relationship('Category',
            backref=db.backref('posts', lazy='dynamic'))

        def __init__(self, title, body, category, pub_date=None):
            self.title = title
            self.body = body
            if pub_date is None:
                pub_date = datetime.utcnow()
            self.pub_date = pub_date
            self.category = category

        def __repr__(self):
            return '<Post %r>' % self.title


    class Category(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(50))

        def __init__(self, name):
            self.name = name

        def __repr__(self):
            return '<Category %r>' % self.name

首先让我们创建一些对象:

>>> py = Category('Python')
>>> p = Post('Hello Python!', 'Python is pretty cool', py)
>>> db.session.add(py)
>>> db.session.add(p)

现在因为我们在 backref 中声明了 `posts` 作为动态关系，查询显示为:

>>> py.posts
<sqlalchemy.orm.dynamic.AppenderBaseQuery object at 0x1027d37d0>

它的行为像一个普通的查询对象，因此我们可以查询与我们测试的 “Python” 分类相关的所有文章(posts):

>>> py.posts.all()
[<Post 'Hello Python!'>]


启蒙之路
---------------------

您仅需要知道与普通的 SQLAlchemy 不同之处:

1.  :class:`SQLAlchemy` 允许您访问下面的东西:

    -   :mod:`sqlalchemy` 和 :mod:`sqlalchemy.orm` 下所有的函数和类
    -   一个叫做 `session` 的预配置范围的会话(session)
    -   :attr:`~SQLAlchemy.metadata` 属性
    -   :attr:`~SQLAlchemy.engine` 属性
    -   :meth:`SQLAlchemy.create_all` 和 :meth:`SQLAlchemy.drop_all`，根据模型用来创建以及删除表格的方法
    -   一个 :class:`Model` 基类，即是一个已配置的声明(declarative)的基础(base)

2.  :class:`Model` 声明基类行为类似一个常规的 Python 类，不过有个 `query` 属性，可以用来查询模型 (:class:`Model` 和 :class:`BaseQuery`)

3.  您必须提交会话，但是没有必要在每个请求后删除它(session)，Flask-SQLAlchemy 会帮您完成删除操作。

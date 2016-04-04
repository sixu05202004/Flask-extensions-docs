.. _binds:

.. currentmodule:: flask.ext.sqlalchemy

绑定多个数据库
=============================

从 0.12 开始，Flask-SQLAlchemy 可以容易地连接到多个数据库。为了实现这个功能，预配置了 SQLAlchemy 来支持多个 “binds”。

什么是绑定(binds)? 在 SQLAlchemy 中一个绑定(bind)是能执行 SQL 语句并且通常是一个连接或者引擎类的东东。在 Flask-SQLAlchemy 中，绑定(bind)总是背后自动为您创建好的引擎。这些引擎中的每个之后都会关联一个短键（bind key）。这个键会在模型声明时使用来把一个模型关联到一个特定引擎。

如果模型没有关联一个特定的引擎的话，就会使用默认的连接(``SQLALCHEMY_DATABASE_URI`` 配置值)。

示例配置
---------------------

下面的配置声明了三个数据库连接。特殊的默认值和另外两个分别名为 `users`（用于用户）和 `appmeta` 连接到一个提供只读访问应用内部数据的 sqlite 数据库）::

    SQLALCHEMY_DATABASE_URI = 'postgres://localhost/main'
    SQLALCHEMY_BINDS = {
        'users':        'mysqldb://localhost/users',
        'appmeta':      'sqlite:////path/to/appmeta.db'
    }

创建和删除表
----------------------------

:meth:`~SQLAlchemy.create_all` 和 :meth:`~SQLAlchemy.drop_all` 方法默认作用于所有声明的绑定(bind)，包括默认的。这个行为可以通过提供 `bind` 参数来定制。它可以是单个绑定(bind)名, ``'__all__'`` 指向所有绑定(binds)或一个绑定(bind)名的列表。默认的绑定(bind)(``SQLALCHEMY_DATABASE_URI``) 名为 `None`:

>>> db.create_all()
>>> db.create_all(bind=['users'])
>>> db.create_all(bind='appmeta')
>>> db.drop_all(bind=None)

引用绑定(Binds)
------------------

当您声明模型时，您可以用 :attr:`~Model.__bind_key__` 属性指定绑定(bind)::

    class User(db.Model):
        __bind_key__ = 'users'
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)

bind key 存储在表中的 `info` 字典中作为 ``'bind_key'`` 键值。了解这个很重要，因为当您想要直接创建一个表对象时，您会需要把它放在那::

    user_favorites = db.Table('user_favorites',
        db.Column('user_id', db.Integer, db.ForeignKey('user.id')),
        db.Column('message_id', db.Integer, db.ForeignKey('message.id')),
        info={'bind_key': 'users'}
    )

如果您在模型上指定了 `__bind_key__` ，您可以用它们准确地做您想要的。模型会自行连 接到指定的数据库连接。


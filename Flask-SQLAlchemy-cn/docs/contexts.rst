.. _contexts:

.. currentmodule:: flask.ext.sqlalchemy

引入上下文
==========================

如果您计划只使用一个应用程序，您大可跳过这一章节。只需要把您的应用程序传给 :class:`SQLAlchemy` 构造函数，一般情况下您就设置好了。然而您想要使用不止一个应用或者在一个函数中动态地创建应用的话，您需要仔细阅读。

如果您在一个函数中定义您的应用，但是 :class:`SQLAlchemy` 对象是全局的，后者如何知道前者了？答案就是 :meth:`~SQLAlchemy.init_app` 函数::

    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    db = SQLAlchemy()

    def create_app():
        app = Flask(__name__)
        db.init_app(app)
        return app

它所做的是准备应用以与 :class:`SQLAlchemy` 共同工作。然而现在不把 :class:`SQLAlchemy` 对象绑定到您的应用。为什么不这么做？ 因为那里可能创建不止一个应用。

那么 :class:`SQLAlchemy` 是如何知道您的应用的？您必须配置一个应用上下文。如果您在一个 Flask 视图函数中进行工作，这会自动实现。但如果您在交互式的 shell 中，您需要手动这么做。(参阅 `创建应用上下文 <http://flask.pocoo.org/docs/appcontext/#creating-an-application-context>`_ )。

简而言之，像这样做:

>>> from yourapp import create_app
>>> app = create_app()
>>> app.app_context().push()

在脚本里面使用 with 声明都样也有作用::

    def my_function():
        with app.app_context():
            user = db.User(...)
            db.session.add(user)
            db.session.commit()

Flask-SQLAlchemy 里的一些函数也可以接受要在其上进行操作的应用作为参数:

>>> from yourapp import db, create_app
>>> db.create_all(app=create_app())

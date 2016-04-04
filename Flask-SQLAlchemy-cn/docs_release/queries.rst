.. currentmodule:: flask.ext.sqlalchemy

选择(Select),插入(Insert), 删除(Delete)
=========================================

现在您已经有了 :ref:`declared models <models>`，是时候从数据库中查询数据。我们将会使用 :ref:`quickstart` 章节中定义的数据模型。


插入记录
-----------------

在查询数据之前我们必须先插入数据。您的所有模型都应该有一个构造函数，如果您 忘记了，请确保加上一个。只有您自己使用这些构造函数而 SQLAlchemy 在内部不会使用它， 所以如何定义这些构造函数完全取决与您。

向数据库插入数据分为三个步骤:

1.  创建 Python 对象
2.  把它添加到会话
3.  提交会话

这里的会话不是 Flask 的会话，而是 Flask-SQLAlchemy 的会话。它本质上是一个 数据库事务的加强版本。它是这样工作的:

>>> from yourapp import User
>>> me = User('admin', 'admin@example.com')
>>> db.session.add(me)
>>> db.session.commit()

好吧，这不是很难吧。但是在您把对象添加到会话之前， SQLAlchemy 基本不考虑把它加到事务中。这是好事，因为您仍然可以放弃更改。比如想象 在一个页面上创建文章，但是您只想把文章传递给模板来预览渲染，而不是把它存进数据库。

调用 :func:`~sqlalchemy.orm.session.Session.add` 函数会添加对象。它会发出一个 `INSERT` 语句给数据库，但是由于事务仍然没有提交，您不会立即得到返回的 ID 。如果您提交，您的用户会有一个 ID:

>>> me.id
1

删除记录
----------------

删除记录是十分类似的，使用 :func:`~sqlalchemy.orm.session.Session.delete` 代替 :func:`~sqlalchemy.orm.session.Session.add`:

>>> db.session.delete(me)
>>> db.session.commit()

查询记录
----------------

那么我们怎么从数据库中查询数据？为此，Flask-SQLAlchemy 在您的 :class:`Model` 类上提供了 :attr:`~Model.query` 属性。当您访问它时，您会得到一个新的所有记录的查询对象。在使用 :func:`~sqlalchemy.orm.query.Query.all` 或者 :func:`~sqlalchemy.orm.query.Query.first` 发起查询之前可以使用方法 :func:`~sqlalchemy.orm.query.Query.filter` 来过滤记录。如果您想要用主键查询的话，也可以使用 :func:`~sqlalchemy.orm.query.Query.get`。

下面的查询假设数据库中有如下条目:

=========== =========== =====================
`id`        `username`  `email`
1           admin       admin@example.com
2           peter       peter@example.org
3           guest       guest@example.com
=========== =========== =====================

通过用户名查询用户:

>>> peter = User.query.filter_by(username='peter').first()
>>> peter.id
1
>>> peter.email
u'peter@example.org'

同上但是查询一个不存在的用户名返回 `None`:

>>> missing = User.query.filter_by(username='missing').first()
>>> missing is None
True

使用更复杂的表达式查询一些用户:

>>> User.query.filter(User.email.endswith('@example.com')).all()
[<User u'admin'>, <User u'guest'>]

按某种规则对用户排序:

>>> User.query.order_by(User.username)
[<User u'admin'>, <User u'guest'>, <User u'peter'>]

限制返回用户的数量:

>>> User.query.limit(1).all()
[<User u'admin'>]

用主键查询用户:

>>> User.query.get(1)
<User u'admin'>


在视图中查询
----------------

当您编写 Flask 视图函数，对于不存在的条目返回一个 404 错误是非常方便的。因为这是一个很常见的问题，Flask-SQLAlchemy 为了解决这个问题提供了一个帮助函数。可以使用 :meth:`~Query.get_or_404` 来代替 :meth:`~sqlalchemy.orm.query.Query.get`，使用 :meth:`~Query.first_or_404` 来代替 :meth:`~sqlalchemy.orm.query.Query.first`。这样会抛出一个 404 错误，而不是返回 `None`::

    @app.route('/user/<username>')
    def show_user(username):
        user = User.query.filter_by(username=username).first_or_404()
        return render_template('show_user.html', user=user)

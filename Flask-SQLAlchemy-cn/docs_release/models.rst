.. _models:

.. currentmodule:: flask.ext.sqlalchemy

声明模型
================

通常下，Flask-SQLAlchemy 的行为就像一个来自 :mod:`~sqlalchemy.ext.declarative` 扩展配置正确的 declarative 基类。因此，我们强烈建议您阅读 SQLAlchemy 文档以获取一个全面的参考。尽管如此，我们这里还是给出了最常用的示例。

需要牢记的事情:

-   您的所有模型的基类叫做 `db.Model`。它存储在您必须创建的 SQLAlchemy 实例上。
    细节请参阅 :ref:`quickstart`。
-   有一些部分在 SQLAlchemy 上是必选的，但是在 Flask-SQLAlchemy 上是可选的。
    比如表名是自动地为您设置好的，除非您想要覆盖它。它是从转成小写的类名派生出来的，即 “CamelCase” 转换为 “camel_case”。

简单示例
--------------

一个非常简单的例子::

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)

        def __init__(self, username, email):
            self.username = username
            self.email = email

        def __repr__(self):
            return '<User %r>' % self.username

用 :class:`~sqlalchemy.Column` 来定义一列。列名就是您赋值给那个变量的名称。如果您想要在表中使用不同的名称，您可以提供一个想要的列名的字符串作为可选第一个参数。主键用 ``primary_key=True`` 标记。可以把多个键标记为主键，此时它们作为复合主键。

列的类型是 :class:`~sqlalchemy.Column` 的第一个参数。您可以直接提供它们或进一步规定（比如提供一个长度）。下面的类型是最常用的:

=================== =====================================
`Integer`           一个整数
`String` (size)     有长度限制的字符串
`Text`              一些较长的 unicode 文本
`DateTime`          表示为 Python
                    :mod:`~datetime.datetime` 对象的
                    时间和日期
`Float`             存储浮点值
`Boolean`           存储布尔值
`PickleType`        存储为一个持久化的 Python 对象
`LargeBinary`       存储一个任意大的二进制数据
=================== =====================================

一对多(one-to-many)关系
-------------------------

最为常见的关系就是一对多的关系。因为关系在它们建立之前就已经声明，您可以使用 字符串来指代还没有创建的类(例如如果 `Person` 定义了一个到 `Article` 的关系，而 `Article` 在文件的后面才会声明)。

关系使用 :func:`~sqlalchemy.orm.relationship` 函数表示。然而外键必须用类 :class:`sqlalchemy.schema.ForeignKey` 来单独声明::

    class Person(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(50))
        addresses = db.relationship('Address', backref='person',
                                    lazy='dynamic')

    class Address(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        email = db.Column(db.String(50))
        person_id = db.Column(db.Integer, db.ForeignKey('person.id'))

``db.relationship()`` 做了什么？这个函数返回一个可以做许多事情的新属性。在本案例中，我们让它指向 Address 类并加载多个地址。它如何知道会返回不止一个地址？因为 SQLALchemy 从您的声明中猜测了一个有用的默认值。 如果您想要一对一关系，您可以把 ``uselist=False`` 传给 :func:`~sqlalchemy.orm.relationship` 。

那么 `backref` 和 `lazy` 意味着什么了？`backref` 是一个在 `Address` 类上声明新属性的简单方法。您也可以使用 ``my_address.person`` 来获取使用该地址(address)的人(person)。`lazy` 决定了 SQLAlchemy 什么时候从数据库中加载数据:

-   ``'select'`` (默认值) 就是说 SQLAlchemy 会使用一个标准的 select 语句必要时一次加载数据。 
-   ``'joined'`` 告诉 SQLAlchemy 使用 `JOIN` 语句作为父级在同一查询中来加载关系。
-   ``'subquery'`` 类似 ``'joined'`` ，但是 SQLAlchemy 会使用子查询。
-   ``'dynamic'`` 在有多条数据的时候是特别有用的。不是直接加载这些数据，SQLAlchemy 会返回一个查询对象，在加载数据前您可以过滤（提取）它们。

您如何为反向引用（backrefs）定义惰性（lazy）状态？使用 :func:`~sqlalchemy.orm.backref` 函数::

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(50))
        addresses = db.relationship('Address',
            backref=db.backref('person', lazy='joined'), lazy='dynamic')

多对多(many-to-many)关系
--------------------------

如果您想要用多对多关系，您需要定义一个用于关系的辅助表。对于这个辅助表， 强烈建议 *不*  使用模型，而是采用一个实际的表::

    tags = db.Table('tags',
        db.Column('tag_id', db.Integer, db.ForeignKey('tag.id')),
        db.Column('page_id', db.Integer, db.ForeignKey('page.id'))
    )

    class Page(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        tags = db.relationship('Tag', secondary=tags,
            backref=db.backref('pages', lazy='dynamic'))

    class Tag(db.Model):
        id = db.Column(db.Integer, primary_key=True)

这里我们配置 `Page.tags` 加载后作为标签的列表，因为我们并不期望每页出现太多的标签。而每个 tag 的页面列表（ `Tag.pages`）是一个动态的反向引用。 正如上面提到的，这意味着您会得到一个可以发起 select 的查询对象。

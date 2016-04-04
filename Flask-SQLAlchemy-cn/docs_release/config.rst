配置
=============

下面是 Flask-SQLAlchemy 中存在的配置值。Flask-SQLAlchemy 从您的 Flask 主配置中加载这些值。 注意其中的一些在引擎创建后不能修改，所以确保尽早配置且不在运行时修改它们。

配置键
------------------

Flask-SQLAlchemy 扩展能够识别的配置键的清单:

.. tabularcolumns:: |p{6.5cm}|p{8.5cm}|

================================== =======================================================
``SQLALCHEMY_DATABASE_URI``        用于连接数据的数据库。例如：

                                   - ``sqlite:////tmp/test.db``
                                   - ``mysql://username:password@server/db``
``SQLALCHEMY_BINDS``               一个映射绑定 (bind) 键到 
                                   SQLAlchemy 连接 URIs 的字典。
                                   更多的信息请参阅 :ref:`binds`。
``SQLALCHEMY_ECHO``                如果设置成 `True`，SQLAlchemy 将会记录所有
                                   发到标准输出(stderr)的语句，这对调试很有帮助。
``SQLALCHEMY_RECORD_QUERIES``      可以用于显式地禁用或者启用查询记录。查询记录
                                   在调试或者测试模式下自动启用。更多信息请参阅
                                   :func:`get_debug_queries`。
``SQLALCHEMY_NATIVE_UNICODE``      可以用于显式地禁用支持原生的 unicode。这是
                                   某些数据库适配器必须的（像在 Ubuntu 某些版本上的 
                                   PostgreSQL），当使用不合适的指定无编码的数据库
                                   默认值时。
``SQLALCHEMY_POOL_SIZE``           数据库连接池的大小。默认是数据库引擎的默认值
                                   （通常是 5）。
``SQLALCHEMY_POOL_TIMEOUT``        指定数据库连接池的超时时间。默认是 10。
``SQLALCHEMY_POOL_RECYCLE``        自动回收连接的秒数。这对 MySQL 是必须的，默认
                                   情况下 MySQL 会自动移除闲置 8 小时或者以上的连接。
                                   需要注意地是如果使用 MySQL 的话，
                                   Flask-SQLAlchemy 会自动地设置这个值为 2 小时。
``SQLALCHEMY_MAX_OVERFLOW``        控制在连接池达到最大值后可以创建的连接数。当这些额外的
                                   连接回收到连接池后将会被断开和抛弃。
``SQLALCHEMY_TRACK_MODIFICATIONS`` 如果设置成 `True` (默认情况)，Flask-SQLAlchemy
                                   将会追踪对象的修改并且发送信号。这需要额外的内存，
                                   如果不必要的可以禁用它。
================================== =======================================================

.. versionadded:: 0.8
   增加 ``SQLALCHEMY_NATIVE_UNICODE``, ``SQLALCHEMY_POOL_SIZE``,
   ``SQLALCHEMY_POOL_TIMEOUT`` 和 ``SQLALCHEMY_POOL_RECYCLE`` 配置键。

.. versionadded:: 0.12
   增加 ``SQLALCHEMY_BINDS`` 配置键。

.. versionadded:: 0.17
   增加 ``SQLALCHEMY_MAX_OVERFLOW`` 配置键。

.. versionadded:: 2.0
   增加 ``SQLALCHEMY_TRACK_MODIFICATIONS`` 配置键。

连接 URI 格式
-----------------

完整连接 URI 格式列表请跳转到 SQLAlchemy 下面的文档(`支持的数据库
<http://www.sqlalchemy.org/docs/core/engines.html>`_)。这里展示了一些常见的连接字符串。 

SQLAlchemy 把一个引擎的源表示为一个连同设定引擎选项的可选字符串参数的 URI。URI 的形式是::

    dialect+driver://username:password@host:port/database

该字符串中的许多部分是可选的。如果没有指定驱动器，会选择默认的（确保在这种情况下 *不* 包含 ``+`` ）。

Postgres::

    postgresql://scott:tiger@localhost/mydatabase

MySQL::

    mysql://scott:tiger@localhost/mydatabase

Oracle::

    oracle://scott:tiger@127.0.0.1:1521/sidname

SQLite (注意开头的四个斜线)::

    sqlite:////absolute/path/to/foo.db

使用自定义的元数据和命名约定
--------------------------------------------

你可以使用一个自定义的 :class:`~sqlalchemy.schema.MetaData` 对象来构造 :class:`SQLAlchemy` 对象。这允许你指定一个 `自定义约束命名约定 
<http://docs.sqlalchemy.org/en/latest/core/constraints.html#constraint-naming-conventions>`_。这样做对数据库的迁移是很重要的。因为 SQL 没有定义一个标准的命名约定，无法保证数据库之间实现是兼容的。你可以自定义命名约定像 SQLAlchemy 文档建议那样::

    from sqlalchemy import MetaData
    from flask import Flask
    from flask.ext.sqlalchemy import SQLAlchemy

    convention = {
        "ix": 'ix_%(column_0_label)s',
        "uq": "uq_%(table_name)s_%(column_0_name)s",
        "ck": "ck_%(table_name)s_%(constraint_name)s",
        "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
        "pk": "pk_%(table_name)s"
    }

    metadata = MetaData(naming_convention=convention)
    db = SQLAlchemy(app, metadata=metadata)


更多关于 :class:`~sqlalchemy.schema.MetaData` 的信息，`请查看官方的文档
<http://docs.sqlalchemy.org/en/latest/core/metadata.html>`_。

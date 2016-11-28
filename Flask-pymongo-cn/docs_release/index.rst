Flask-PyMongo
=============

`MongoDB <http://www.mongodb.org/>`_ 是一个开源的数据库，它存储着灵活的类-JSON 的“文档”。与关系数据库中的数据行相反，它能够存储任何的数字，名称，或者复杂的层级结构。Python 开发者可以考虑把 MongoDB 作为一个持久化，可搜索的 Python 字典的“仓库”(实际上，这是如何用 `PyMongo
<http://api.mongodb.org/python/current/>`_ 来表示 MongoDB 中的“文档”)。

Flask-PyMongo 架起来 Flask 和 PyMongo 之间的桥梁，因此你能够使用 Flask 正常的机制去配置和连接 MongoDB。


快速入门
----------

首先，安装 Flask-PyMongo:

.. code-block:: bash

    $ pip install Flask-PyMongo

Flask-PyMongo 的各种依赖，比如，最新版本的 Flask（0.8或者以上）以及 PyMongo（2.4或者以上） ，也会为你安装的。Flask-PyMongo 是兼容 Python 2.6, 2.7, 和 3.3 版本并且通过测试。

接着，在你的代码中添加一个 :class:`~flask_pymongo.PyMongo`:

.. code-block:: python

    from flask import Flask
    from flask.ext.pymongo import PyMongo

    app = Flask(__name__)
    mongo = PyMongo(app)

:class:`~flask_pymongo.PyMongo` 连接运行在本机上且端口为 27017 的 MongoDB 服务器，并且假设默认的数据库名为 ``app.name`` (换而言之，你可以使用传入到 :class:`~flask.Flask` 中的任何数据库名)。这个数据库能够作为 :attr:`~flask_pymongo.PyMongo.db` 属性被导入。

你可以在视图中直接使用 :attr:`~flask_pymongo.PyMongo.db` :

.. code-block:: python

    @app.route('/')
    def home_page():
        online_users = mongo.db.users.find({'online': True})
        return render_template('index.html',
            online_users=online_users)


Helpers
-------

Flask-PyMongo 提供一些通用任务的现成方法:

.. automethod:: flask_pymongo.wrappers.Collection.find_one_or_404

.. automethod:: flask_pymongo.PyMongo.send_file

.. automethod:: flask_pymongo.PyMongo.save_file

.. autoclass:: flask_pymongo.BSONObjectIdConverter

Configuration
-------------

:class:`~flask_pymongo.PyMongo` 直接支持如下的配置项:

============================ ===================================================
``MONGO_URI``                一个 `MongoDB 网址
                             <http://www.mongodb.org/display/DOCS/Connections#Connections-StandardConnectionStringFormat>`_
                             用于其他配置项。
``MONGO_HOST``               你的 MongoDB 服务器的主机名或者 IP 地址。 
                             默认：”localhost”。
``MONGO_PORT``               你的 MongoDB 服务器的端口。默认：27017。
``MONGO_AUTO_START_REQUEST`` 为了禁用 PyMongo 2.2 的 “auto start request” 行为，
                             设置成 ``False``。 
                             (请见 :class:`~pymongo.mongo_client.MongoClient`)。 
                             默认：``True``。
``MONGO_MAX_POOL_SIZE``      (可选): PyMongo 连接池中保持空闲连接的最大数量。 
                             默认：PyMongo 默认值。
``MONGO_SOCKET_TIMEOUT_MS``  (可选): (整型) 在超时前套接字允许一个发送或者接收的耗时(毫秒)。 
                             默认: PyMongo 默认值。
``MONGO_CONNECT_TIMEOUT_MS`` (可选): (整型) 在超时前允许一个连接的耗时(毫秒)。 
                             默认: PyMongo 默认值。
``MONGO_DBNAME``             可用于作为 ``db`` 属性的数据库名。默认： ``app.name``。
``MONGO_USERNAME``           用于认证的用户名。默认：``None``。
``MONGO_PASSWORD``           用于认证的密码。默认：``None``。
``MONGO_REPLICA_SET``        设置成连接的备份集的名称；
                             这必须匹配到备份集的内部名，由 `<http://www.mongodb.org/display/DOCS/Replica+Set+Commands#ReplicaSetCommands-isMaster>`_ 命令)决定的。默认：``None``。
``MONGO_READ_PREFERENCE``    决定如何读取路由到备份集的成员。
                             必须是定义在 :class:`pymongo.read_preferences.ReadPreference` 中的一个常量 或者一个字符串名称。 
``MONGO_DOCUMENT_CLASS``     告诉 pymongo 返回定制的对象而不是默认的字典，
                             比如 ``bson.son.SON``。 默认： ``dict``。
============================ ===================================================

当 :class:`~flask_pymongo.PyMongo` 或者 :meth:`~flask_pymongo.PyMongo.init_app` 仅仅只有一个参数调用的时候 (the Flask 实例)，会假设配置值的前缀是 ``MONGO``；能够用 `config_prefix` 来覆盖这个前缀。

这个技术能够用于连接多个数据库或者数据服务器:

.. code-block:: python

    app = Flask(__name__)

    # connect to MongoDB with the defaults
    mongo1 = PyMongo(app)

    # connect to another MongoDB database on the same host
    app.config['MONGO2_DBNAME'] = 'dbname_two'
    mongo2 = PyMongo(app, config_prefix='MONGO2')

    # connect to another MongoDB server altogether
    app.config['MONGO3_HOST'] = 'another.host.example.com'
    app.config['MONGO3_PORT'] = 27017
    app.config['MONGO3_DBNAME'] = 'dbname_three'
    mongo3 = PyMongo(app, config_prefix='MONGO3')

你应该需要注意一些自动配置的设置:

``tz_aware``:
  Flask-PyMongo 一直使用通用时区的 :class:`~datetime.datetime` 对象。这是因为当建立连接的时候它设置 ``tz_aware`` 参数为 ``True``。从 MongoDB 返回的 :class:`~datetime.datetime` 对象一直是 UTC。

``safe``:
  Flask-PyMongo 默认地设置成 “safe” 模式，这会导致
  :meth:`~pymongo.collection.Collection.save`,
  :meth:`~pymongo.collection.Collection.insert`,
  :meth:`~pymongo.collection.Collection.update`, 和
  :meth:`~pymongo.collection.Collection.remove` 在返回前一直等待着服务器的应答。你可以在调用的时候通过传入 ``safe=False`` 参数到任何一个受影响的方法中来覆盖它。


API
===

常量
---------

.. autodata:: flask_pymongo.ASCENDING

.. autodata:: flask_pymongo.DESCENDING


类
-------

.. autoclass:: flask_pymongo.PyMongo
   :members:

.. autoclass:: flask_pymongo.wrappers.Collection
   :members:


封装
--------

These classes exist solely in order to make expressions such as
``mongo.db.foo.bar`` evaluate to a
:class:`~flask_pymongo.wrappers.Collection` instance instead of
a :class:`pymongo.collection.Collection` instance. They are documented here
solely for completeness.

.. autoclass:: flask_pymongo.wrappers.MongoClient
   :members:

.. autoclass:: flask_pymongo.wrappers.MongoReplicaSetClient
   :members:

.. autoclass:: flask_pymongo.wrappers.Database
   :members:


历史和贡献
------------------------

更新:
- 0.4.0: October 19, 2015

  - Flask-Pymongo is now compatible with pymongo 3.0+:
    `#63 <https://github.com/dcrosta/flask-pymongo/pull/63>`_.

- 0.3.1: April 9, 2015

  - Flask-PyMongo is now tested against Python 2.6, 2.7, 3.3, and 3.4.
  - Flask-PyMongo installation now no longer depends on `nose
    <https://pypi.python.org/pypi/nose/>`_.
  - `#58 <https://github.com/dcrosta/flask-pymongo/pull/58>`_ Update
    requirements for PyMongo 3.x (Emmanuel Valette).
  - `#43 <https://github.com/dcrosta/flask-pymongo/pull/43>`_ Ensure error
    is raised when URI database name is parsed as 'None' (Ben Jeffrey).
  - `#50 <https://github.com/dcrosta/flask-pymongo/pull/50>`_ Fix a bug in
    read preference handling (Kevin Funk).
  - `#46 <https://github.com/dcrosta/flask-pymongo/issues/46>`_ Cannot use
    multiple replicaset instances which run on different ports (Mark
    Unsworth).
  - `#30 <https://github.com/dcrosta/flask-pymongo/issues/30>`_
    ConfiguationError with MONGO_READ_PREFERENCE (Mark Unsworth).

- 0.3.0: July 4, 2013

  - This is a minor version bump which introduces backwards breaking
    changes! Please read these change notes carefully.
  - Removed read preference constants from Flask-PyMongo; to set a
    read preference, use the string name or import contants directly
    from :class:`pymongo.read_preferences.ReadPreference`.
  - `#22 (partial) <https://github.com/dcrosta/flask-pymongo/pull/22>`_
    Add support for ``MONGO_SOCKET_TIMEOUT_MS`` and
    ``MONGO_CONNECT_TIMEOUT_MS`` options (ultrabug).
  - `#27 (partial) <https://github.com/dcrosta/flask-pymongo/pull/27>`_
    Make Flask-PyMongo compatible with Python 3 (Vizzy).

- 0.2.1: December 22, 2012

  - `#19 <https://github.com/dcrosta/flask-pymongo/pull/19>`_ Added
    ``MONGO_DOCUMENT_CLASS`` config option (jeverling).

- 0.2.0: December 15, 2012

  - This is a minor version bump which may introduce backwards breaking
    changes! Please read these change notes carefully.
  - `#17 <https://github.com/dcrosta/flask-pymongo/pull/17>`_ Now using
    PyMongo 2.4's ``MongoClient`` and ``MongoReplicaSetClient`` objects
    instead of ``Connection`` and ``ReplicaSetConnection`` classes
    (tang0th).
  - `#17 <https://github.com/dcrosta/flask-pymongo/pull/17>`_ Now requiring
    at least PyMongo version 2.4 (tang0th).
  - `#17 <https://github.com/dcrosta/flask-pymongo/pull/17>`_ The wrapper
    class ``flask_pymongo.wrappers.Connection`` is renamed to
    ``flask_pymongo.wrappers.MongoClient`` (tang0th).
  - `#17 <https://github.com/dcrosta/flask-pymongo/pull/17>`_ The wrapper
    class ``flask_pymongo.wrappers.ReplicaSetConnection`` is renamed to
    ``flask_pymongo.wrappers.MongoReplicaSetClient`` (tang0th).
  - `#18 <https://github.com/dcrosta/flask-pymongo/issues/18>`_
    ``MONGO_AUTO_START_REQUEST`` now defaults to ``False`` when
    connecting using a URI.

- 0.1.4: December 15, 2012

  - `#15 <https://github.com/dcrosta/flask-pymongo/pull/15>`_ Added support
    for ``MONGO_MAX_POOL_SIZE`` (Fabrice Aneche)

- 0.1.3: September 22, 2012

  - Added support for configuration from MongoDB URI.

- 0.1.2: June 18, 2012

  - Updated wiki example application
  - `#14 <https://github.com/dcrosta/flask-pymongo/issues/14>`_ Added
    examples and docs to PyPI package.

- 0.1.1: May 26, 2012

  - Added support for PyMongo 2.2's "auto start request" feature, by way
    of the ``MONGO_AUTO_START_REQUEST`` configuration flag.
  - `#13 <https://github.com/dcrosta/flask-pymongo/pull/13>`_ Added
    BSONObjectIdConverter (Christoph Herr)
  - `#12 <https://github.com/dcrosta/flask-pymongo/pull/12>`_ Corrected
    documentation typo (Thor Adam)

- 0.1: December 21, 2011

  - Initial Release


贡献者:

- `jeverling <https://github.com/jeverling>`_
- `tang0th <https://github.com/tang0th>`_
- `Fabrice Aneche <https://github.com/akhenakh>`_
- `Thor Adam <https://github.com/thoradam>`_
- `Christoph Herr <https://github.com/jarus>`_
- `Mark Unsworth <https://github.com/markunsworth>`_
- `Kevin Funk <https://github.com/k-funk>`_
- `Ben Jeffrey <https://github.com/jeffbr13>`_
- `Emmanuel Valette <https://github.com/karec>`_

Flask-Cache
================

.. module:: flask.ext.cache

安装
------------

使用下面的命令行安装 Flask-Cache::

    $ easy_install Flask-Cache

或者可以用下面的命令行，如果安装了 pip::

    $ pip install Flask-Cache

使用
------

缓存（Cache）是通过使用一个 ``Cache`` 实例进行管理::

    from flask import Flask
    from flask.ext.cache import Cache

    app = Flask(__name__)
    # Check Configuring Flask-Cache section for more details
    cache = Cache(app,config={'CACHE_TYPE': 'simple'})

你能够用 **init_app** 方法在初始化 ``Cache`` 后设置它::

    cache = Cache(config={'CACHE_TYPE': 'simple'})

    app = Flask(__name__)
    cache.init_app(app)

如果有多个 ``Cache`` 实例以及每一个实例都有不同后端的话（换句话说，就是每一个实例使用不用的缓存类型CACHE_TYPE），使用配置字典是十分有用的::

    #: Method A: During instantiation of class
    cache = Cache(config={'CACHE_TYPE': 'simple'})
    #: Method B: During init_app call
    cache.init_app(app, config={'CACHE_TYPE': 'simple'})

.. versionadded:: 0.7

缓存视图函数
----------------------

使用装饰器 :meth:`~Cache.cached` 能够缓存视图函数。它在默认情况下使用请求路径(request.path)作为cache_key::

    @cache.cached(timeout=50)
    def index():
        return render_template('index.html')

该装饰器有一个可选的参数：``unless``，它允许一个可调用的、返回值是True或者False的函数。如果 ``unless`` 返回
``True``，将会完全忽略缓存机制（内置的缓存机制会完全不起作用）。

缓存其它函数
-----------------------

同样地，使用 ``@cached`` 装饰器也能够缓存其它非视图函数的结果。唯一的要求是需要指定 ``key_prefix``，否则会使用请求路径(request.path)作为cache_key::

    @cache.cached(timeout=50, key_prefix='all_comments')
    def get_all_comments():
        comments = do_serious_dbio()
        return [x.author for x in comments]

    cached_comments = get_all_comments()

Memoization（一种缓存技术）
---------------------------

请参看 :meth:`~Cache.memoize`

在memoization中，函数参数同样包含cache_key。

.. Note::

	如果函数不接受参数的话，:meth:`~Cache.cached` 和
	:meth:`~Cache.memoize` 两者的作用是一样的。

Memoize同样也为类成员函数而设计，因为它根据 `identity <http://docs.python.org/library/functions.html#id>`_ 将 'self' 或者 'cls' 参数考虑进作为缓存键的一部分。

memoization背后的理论是：在一次请求中如果一个函数需要被调用多次，它只会计算第一次使用这些参数调用该函数。例如，存在一个决定用户角色的 sqlalchemy对象，在一个请求中可能需要多次调用这个函数。为了避免每次都从数据库获取信息，你可以这样做::

    class Person(db.Model):
    	@cache.memoize(50)
    	def has_membership(self, role_id):
    		return Group.query.filter_by(user=self, role_id=role_id).count() >= 1


.. Warning::

    使用可变对象（例如类）作为缓存键的一部分是十分棘手的。建议最好不要让一个对象的实例成为一个memoized函数。然而，memoize在处理参数的时候会执行repr()，因此如果一个对象有__repr__函数，并且返回一个唯一标识该对象的字符串，它将能够作为缓存键的一部分。

    例如，一个sqlalchemy person对象，它返回数据库的ID作为唯一标识符的一部分::

        class Person(db.Model):
            def __repr__(self):
                return "%s(%s)" % (self.__class__.__name__, self.id)

删除memoize的缓存
``````````````````````

.. versionadded:: 0.2

在每个函数的基础上，您可能需要删除缓存。使用上面的例子，让我们来改变用户权限，并将它们分配到一个角色，如果它们新拥有或者失去某些成员关系，现在你需要重新计算。你能够用 :meth:`~Cache.delete_memoized` 函数来达到目的::

	cache.delete_memoized('user_has_membership')

.. Note::

  如果仅仅只有函数名作为参数，所有的memoized的版本将会无效的。然而，您可以删除特定的缓存提供缓存时相同的参数值。在下面的例子中，只有 ``user`` 角色缓存被删除：

  .. code-block:: python

     user_has_membership('demo', 'admin')
     user_has_membership('demo', 'user')

     cache.delete_memoized('user_has_membership', 'demo', 'user')

缓存Jinja2片段
-----------------------

用法::

    {% cache [timeout [,[key1, [key2, ...]]]] %}
    ...
    {% endcache %}

默认情况下“模版文件路径”+“片段开始的函数”用来作为缓存键。同样键名是可以手动设置的。键名串联成一个字符串，这样能够用于避免同样的块在不同模版被重复计算。 

设置 timeout 为 None，并且使用了自定义的键::

    {% cache None "key" %}...
    
为了删除缓存值，为“del”设置超时时间::

    {% cache 'del' %}...

如果提供键名，你可以很容易地产生模版的片段密钥，从模板上下文外删除它::

    from flask.ext.cache import make_template_fragment_key
    key = make_template_fragment_key("key1", vary_on=["key2", "key3"])
    cache.delete(key)

例子::

    Considering we have render_form_field and render_submit macroses.
    {% cache 60*5 %}
    <div>
        <form>
        {% render_form_field form.username %}
        {% render_submit %}
        </form>
    </div>
    {% endcache %}

清除缓存
--------------

请参看 :meth:`~Cache.clear`.

下面的例子是一个用来清空应用缓存的脚本：

.. code-block:: python

    from flask.ext.cache import Cache

    from yourapp import app, your_cache_config

    cache = Cache()


    def main():
        cache.init_app(app, config=your_cache_config)

        with app.app_context():
            cache.clear()

    if __name__ == '__main__':
        main()


.. Warning::

    某些缓存类型不支持完全清空缓存。同样，如果你不使用键前缀，一些缓存类型将刷新整个数据库。请确保你没有任何其他数据存储在缓存数据库中。

配置Flask-Cache
-----------------------

Flask-Cache有下面一些配置项:

.. tabularcolumns:: |p{6.5cm}|p{8.5cm}|


=============================== ==================================================================
``CACHE_TYPE``                  指定哪些类型的缓存对象来使用。
                                这是一个输入字符串，将被导入并实例化。
                                它假设被导入的对象是一个依赖于werkzeug缓存API，
                                返回缓存对象的函数。

                                对于werkzeug.contrib.cache对象，不必给出完整的字符串，
                                只要是下列这些名称之一。

                                内建缓存类型：

                                * **null**: NullCache (default)
                                * **simple**: SimpleCache
                                * **memcached**: MemcachedCache (pylibmc or memcache required)
                                * **gaememcached**: GAEMemcachedCache
                                * **redis**: RedisCache (Werkzeug 0.7 required)
                                * **filesystem**: FileSystemCache
                                * **saslmemcached**: SASLMemcachedCache (pylibmc required)

``CACHE_NO_NULL_WARNING``       当使用的缓存类型是'null'，不会抛出警告信息。
``CACHE_ARGS``                  可选的列表，在缓存类实例化的时候会对该列表进行拆分以及传递（传参）。
``CACHE_OPTIONS``               可选的字典，在缓存类实例化的时候会传递该字典（传参）。
``CACHE_DEFAULT_TIMEOUT``       如果没有设置延迟时间，默认的延时时间会被使用。单位为秒。
``CACHE_THRESHOLD``             最大的缓存条目数，超过该数会删除一些缓存条目。仅仅用于SimpleCache和
                                FileSystemCache。
``CACHE_KEY_PREFIX``            所有键之前添加的前缀。
                                这使得它可以为不同的应用程序使用相同的memcached服务器。
                                仅仅用于RedisCache，MemcachedCache以及GAEMemcachedCache。
``CACHE_MEMCACHED_SERVERS``     服务器地址列表或元组。仅用于MemcachedCache。
``CACHE_MEMCACHED_USERNAME``    SASL与memcached服务器认证的用户名。
                                仅用于SASLMemcachedCache。
``CACHE_MEMCACHED_PASSWORD``    SASL与memcached服务器认证的密码。
                                仅用于SASLMemcachedCache。
``CACHE_REDIS_HOST``            Redis服务器的主机。仅用于RedisCache。
``CACHE_REDIS_PORT``            Redis服务器的端口。默认是6379。仅用于RedisCache。
``CACHE_REDIS_PASSWORD``        用于Redis服务器的密码。仅用于RedisCache。
``CACHE_REDIS_DB``              Redis的db库 (基于零号索引)。默认是0。仅用于RedisCache。
``CACHE_DIR``                   存储缓存的目录。仅用于FileSystemCache。
``CACHE_REDIS_URL``             连接到Redis服务器的URL。
                                例如：``redis://user:password@localhost:6379/2``。
                                仅用于RedisCache。
=============================== ==================================================================


此外，如果标准的Flask配置项 ``TESTING`` 使用并且设置为True的话， **Flask-Cache** 将只会使用NullCache作为缓存类型。

内建的缓存类型
-----------------------

NullCache -- null
`````````````````

不缓存内容

- CACHE_ARGS
- CACHE_OPTIONS


SimpleCache -- simple
`````````````````````

使用本地Python字典缓存。这不是真正的线程安全。

相关配置

- CACHE_DEFAULT_TIMEOUT
- CACHE_THRESHOLD
- CACHE_ARGS
- CACHE_OPTIONS

FileSystemCache -- filesystem
`````````````````````````````

使用文件系统来存储缓存值

- CACHE_DEFAULT_TIMEOUT
- CACHE_DIR
- CACHE_THRESHOLD
- CACHE_ARGS
- CACHE_OPTIONS

MemcachedCache -- memcached
```````````````````````````

使用memcached服务器作为后端。支持pylibmc或memcache或谷歌应用程序引擎的memcache库。

相关配置项

- CACHE_DEFAULT_TIMEOUT
- CACHE_KEY_PREFIX
- CACHE_MEMCACHED_SERVERS
- CACHE_ARGS
- CACHE_OPTIONS

GAEMemcachedCache -- gaememcached
`````````````````````````````````

MemcachedCache一个不同的名称

SASLMemcachedCache -- saslmemcached
```````````````````````````````````

使用memcached服务器作为后端。使用SASL建立与memcached服务器的连接。pylibmc是必须的，libmemcached必须支持SASL。

相关配置项

- CACHE_DEFAULT_TIMEOUT
- CACHE_KEY_PREFIX
- CACHE_MEMCACHED_SERVERS
- CACHE_MEMCACHED_USERNAME
- CACHE_MEMCACHED_PASSWORD
- CACHE_ARGS
- CACHE_OPTIONS

.. versionadded:: 0.10

SpreadSASLMemcachedCache -- spreadsaslmemcachedcache
````````````````````````````````````````````````````

与SASLMemcachedCache一样，但是如果大于memcached的传输安全性，默认是1M，能够跨不同的键名缓存值。使用pickle模块。

.. versionadded:: 0.11

RedisCache -- redis
```````````````````

- CACHE_DEFAULT_TIMEOUT
- CACHE_KEY_PREFIX
- CACHE_REDIS_HOST
- CACHE_REDIS_PORT
- CACHE_REDIS_PASSWORD
- CACHE_REDIS_DB
- CACHE_ARGS
- CACHE_OPTIONS
- CACHE_REDIS_URL


定制缓存后端（后台）
---------------------

你能够轻易地定制缓存后端，只需要导入一个能够实例化以及返回缓存对象的函数。``CACHE_TYPE`` 将是你自定义的函数名的字符串。 这个函数期望得到三个参数。

* ``app``
* ``args``
* ``kwargs``

你自定义的缓存对象必须是
:class:`werkzeug.contrib.cache.BaseCache` 的子类。确保 ``threshold`` 是包含在kwargs参数中，因为它是所有BaseCache类通用的。

Redis的缓存实现的一个例子::

    #: the_app/custom.py
    class RedisCache(BaseCache):
        def __init__(self, servers, default_timeout=500):
            pass

    def redis(app, config, args, kwargs):
       args.append(app.config['REDIS_SERVERS'])
       return RedisCache(*args, **kwargs)

在这个例子中，``CACHE_TYPE`` 可能就是 ``the_app.custom.redis``。

PylibMC缓存实现的一个例子::

    #: the_app/custom.py
    def pylibmccache(app, config, args, kwargs):
        return pylibmc.Client(servers=config['CACHE_MEMCACHED_SERVERS'],
                              username=config['CACHE_MEMCACHED_USERNAME'],
                              password=config['CACHE_MEMCACHED_PASSWORD'],
                              binary=True)

在这个例子中，``CACHE_TYPE`` 可能就是 ``the_app.custom.pylibmccache``。

API
---

.. autoclass:: Cache
   :members: init_app,
             get, set, add, delete, get_many, set_many, delete_many, clear,
             cached, memoize, delete_memoized, delete_memoized_verhash

.. include:: ../CHANGES

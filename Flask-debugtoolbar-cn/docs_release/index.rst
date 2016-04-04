Flask-DebugToolbar
==================

该扩展为 Flask 应用程序添加了一个包含有用的调试信息的工具栏。

.. image:: _static/example.gif

安装
------------

简单地使用 `pip`_ 来安装::

    $ pip install flask-debugtoolbar

.. _pip: https://pip.pypa.io/


用法
-----

设置调试工具栏是简单的::

    from flask import Flask
    from flask_debugtoolbar import DebugToolbarExtension

    app = Flask(__name__)

    # the toolbar is only enabled in debug mode:
    app.debug = True

    # set a 'SECRET_KEY' to enable the Flask session cookies
    app.config['SECRET_KEY'] = '<replace with a secret key>'

    toolbar = DebugToolbarExtension(app)


当调试模式开启的时候，工具栏会自动地给添加到 Jinja 模板中。在生产环境中，设置 ``app.debug = False`` 将会禁用工具栏。

该扩展也支持 Flask 应用的工厂模式，先单独地创建工具栏接着后面为应用初始化它::

    toolbar = DebugToolbarExtension()
    # Then later on.
    app = create_app('the-config.cfg')
    toolbar.init_app(app)

配置
-------------

工具栏支持多个配置选项:

====================================  =====================================   ==========================
名称                                   描述                                     默认值
====================================  =====================================   ==========================
``DEBUG_TB_ENABLED``                  启用工具栏？                              ``app.debug``
``DEBUG_TB_HOSTS``                    显示工具栏的 hosts 白名单                  任意 host
``DEBUG_TB_INTERCEPT_REDIRECTS``      要拦截重定向？                            ``True``
``DEBUG_TB_PANELS``                   面板的模板/类名的清单                      允许所有内置的面板
``DEBUG_TB_PROFILER_ENABLED``         启用所有请求的分析工具                     ``False``, 用户自行开启
``DEBUG_TB_TEMPLATE_EDITOR_ENABLED``  启用模板编辑器                            ``False``
====================================  =====================================   ==========================

要更改配置选项之一，在 Flask 应用程序配置中像这样设置它::

    app.config['DEBUG_TB_INTERCEPT_REDIRECTS'] = False


面板
------

.. toctree::

   panels

贡献
------------

Fork 我们在 `GitHub <https://github.com/mgood/flask-debugtoolbar>`_  上。

感谢
------

本扩展是基于 `django-debug-toolbar`_ 。多谢 `Michael van Tellingen`_ 为了这个 Flask 扩展最初的开发，并且感谢 `individual contributors`_ 中的每一个人。

.. _django-debug-toolbar: https://github.com/django-debug-toolbar/django-debug-toolbar
.. _Michael van Tellingen: https://github.com/mvantellingen
.. _individual contributors: https://github.com/mgood/flask-debugtoolbar/graphs/contributors

索引和表格
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`


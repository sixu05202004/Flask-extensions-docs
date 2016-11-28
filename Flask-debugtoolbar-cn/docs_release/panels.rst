内置的 Panels
===============

版本
--------
    flask_debugtoolbar.panels.versions.VersionDebugPanel

显示已安装的 Flask 版本。展开的视图显示了由 ``setuptools`` 发现的所有已安装的包和它们的版本。


时间
----

    flask_debugtoolbar.panels.timer.TimerDebugPanel

显示处理当前请求的时间。展开后的视图包含了用户态和系统态，执行时间，上下文切换的 CPU 时间分解。

.. image:: _static/screenshot-time-panel.png


HTTP 头
------------

    flask_debugtoolbar.panels.headers.HeaderDebugPanel

显示了目前请求的 HTTP 头。

.. image:: _static/screenshot-headers-panel.png


Request 变量
------------

    flask_debugtoolbar.panels.request_vars.RequestVarsDebugPanel

展现了 Flask 请求相关的变量的细节，包含视图函数变量，会话变量，以及 GET 和 POST 变量。

.. image:: _static/screenshot-request-vars-panel.png


配置
------

    flask_debugtoolbar.panels.config_vars.ConfigVarsDebugPanel

显示了 Flask 应用程序配置字典 ``app.config`` 的内容。

.. image:: _static/screenshot-config-panel.png


模板
---------

    flask_debugtoolbar.panels.template.TemplateDebugPanel

显示关于为某个请求渲染模板的信息，以及提供的模板参数的值。

.. image:: _static/screenshot-template-panel.png


SQLAlchemy
----------

    flask_debugtoolbar.panels.sqlalchemy.SQLAlchemyDebugPanel

显示了当前请求过程中运行的 SQL 查询。

.. note:: 为了记录查询，这个面板需要使用 `Flask-SQLAlchemy`_ 扩展。
   请查看 Flask-SQLAlchemy 的 :ref:`flasksqlalchemy:quickstart` 
   章节来配置它。

   更多关于查询记录的信息请查看文档 :py:func:`~flask.ext.sqlalchemy.get_debug_queries`。


.. image:: _static/screenshot-sqlalchemy-panel.png

.. _Flask-SQLAlchemy: http://flask-sqlalchemy.pocoo.org/


日志
-------

    flask_debugtoolbar.panels.logger.LoggingPanel

显示了当前请求的日志信息

.. image:: _static/screenshot-logger-panel.png


路由列表
----------

    flask_debugtoolbar.panels.route_list.RouteListDebugPanel


显示了 Flask URL 路由规则。


分析/探查
--------

    flask_debugtoolbar.panels.profiler.ProfilerDebugPanel

报告当前请求的分析/探查数据。由于性能的考虑，默认情况下分析/探查是禁用的。单击点中选择分析/探查的标记来决定开启或者关闭。在启用分析/探查后，重新刷新页面来运行分析/探查。

.. image:: _static/screenshot-profiler-panel.png

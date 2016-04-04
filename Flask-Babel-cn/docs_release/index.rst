Flask-Babel
===========

.. module:: flask.ext.babel

Flask-Babel 是一个 `Flask`_ 的扩展，在 `babel`_, `pytz`_ 和
`speaklater`_ 的帮助下添加 i18n 和 l10n 支持到任何 Flask 应用。它内置了一个时间格式化的支持，同样内置了一个非常简单和友好的 :mod:`gettext` 翻译的接口。

安装
------------

下面命令可以安装扩展::

    $ easy_install Flask-Babel

或者如果你安装了 pip::

    $ pip install Flask-Babel

请注意 Flask-Babel 需要 Jinja 2.5。如果你安装一个老的版本你将会需要升级或者禁止 Jinja 支持。


配置
-------------

在配置好应用后所有需要做的就是实例化一个 :class:`Babel` 对象::

    from flask import Flask
    from flask.ext.babel import Babel

    app = Flask(__name__)
    app.config.from_pyfile('mysettings.cfg')
    babel = Babel(app)

babel 对象本身以后支持用于配置 babel。Babel 有两个配置值，这两个配置值能够改变内部的默认值:

=========================== =============================================
`BABEL_DEFAULT_LOCALE`      如果没有指定地域且选择器已经注册，
                            默认是缺省地域。默认是 ``'en'``。
`BABEL_DEFAULT_TIMEZONE`    用户默认使用的时区。默认是 ``'UTC'``。选用默
                            认值的时候，你的应用内部必须使用该时区。
=========================== =============================================

对于更复杂的应用你可能希望对于不同的用户有多个应用，这个时候是选择器函数派上用场的时候。babel 扩展第一次需要当前用户的地区的时候，它会调用 :meth:`~Babel.localeselector` 函数，第一次需要时区的时候，它会调用 :meth:`~Babel.timezoneselector` 函数。 

如果这些方法的任何一个返回 `None`，扩展将会自动回落到配置中的值。而且为了效率考虑函数只会调用一次并且返回值会被缓存。如果你需要在一个请求中切换语言的话，你可以 :func:`refresh` 缓存。

选择器函数的例子::

    from flask import g, request

    @babel.localeselector
    def get_locale():
        # if a user is logged in, use the locale from the user settings
        user = getattr(g, 'user', None)
        if user is not None:
            return user.locale
        # otherwise try to guess the language from the user accept
        # header the browser transmits.  We support de/fr/en in this
        # example.  The best match wins.
        return request.accept_languages.best_match(['de', 'fr', 'en'])

    @babel.timezoneselector
    def get_timezone():
        user = getattr(g, 'user', None)
        if user is not None:
            return user.timezone

以上的例子假设当前的用户是存储在 :data:`flask.g` 对象中。


格式化日期
----------------

你可以使用 :func:`format_datetime`，:func:`format_date`，:func:`format_time` 以及 :func:`format_timedelta` 函数来格式化日期。它们都接受一个 :class:`datetime.datetime` （或者 :class:`datetime.date`，:class:`datetime.time` 以及 :class:`datetime.timedelta`）对象作为第一个参数，其它参数是一个可选的格式化字符串。应用程序应该使用天然的 datetime 对象且内部使用 UTC 作为默认时区。格式化的时候会自动地转换成用户时区以防它不同于 UTC。


为了能够在命令行中使用日期格式化，你可以使用 :meth:`~flask.Flask.test_request_context` 方法:

>>> app.test_request_context().push()

这里是一些例子:

>>> from flask.ext.babel import format_datetime
>>> from datetime import datetime
>>> format_datetime(datetime(1987, 3, 5, 17, 12))
u'Mar 5, 1987 5:12:00 PM'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'full')
u'Thursday, March 5, 1987 5:12:00 PM World (GMT) Time'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'short')
u'3/5/87 5:12 PM'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'dd mm yyy')
u'05 12 1987'
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'dd mm yyyy')
u'05 12 1987'

接着用不同的语言再次格式化:

>>> app.config['BABEL_DEFAULT_LOCALE'] = 'de'
>>> from flask.ext.babel import refresh; refresh()
>>> format_datetime(datetime(1987, 3, 5, 17, 12), 'EEEE, d. MMMM yyyy H:mm')
u'Donnerstag, 5. M\xe4rz 1987 17:12'

关于格式例子的更多信息请参阅 `babel`_ 文档。


使用翻译
------------------

日期格式化之外的另一个部分就是翻译。Flask 使用 :mod:`gettext` 和 Babel 配合一起实现翻译的功能。gettext 的作用就是你可以标记某些字符串作为翻译的内容并且一个工具会从应用中挑选这些，接着把它们放入一个单独的文件为你来翻译。在运行的时候原始的字符串（应该是英语）将会被你选择的语言替换掉。

有两个函数可以用来完成翻译：:func:`gettext` 和 :func:`ngettext`。第一个函数用于翻译含有 0 个或者 1 个字符串参数的字符串，第二个参数用于翻译含有多个字符串参数的字符串。这里有些示例::

    from flask.ext.babel import gettext, ngettext

    gettext(u'A simple string')
    gettext(u'Value: %(value)s', value=42)
    ngettext(u'%(num)s Apple', u'%(num)s Apples', number_of_apples)

另外如果你希望在你的应用中使用常量字符串并且在请求之外定义它们的话，你可以使用一个“懒惰”字符串。“懒惰”字符串直到它们实际被使用的时候才会计算。为了使用一个“懒惰”字符串，请使用 :func:`lazy_gettext` 函数::

    from flask.ext.babel import lazy_gettext

    class MyForm(formlibrary.FormBase):
        success_message = lazy_gettext(u'The form was successfully saved.')

Flask-Babel 如何找到翻译？首先你必须要生成翻译。这里是你如何做到这一点:


翻译应用
------------------------

首先你需要用 :func:`gettext` 或者 :func:`ngettext` 在你的应用中标记你要翻译的所有字符串。在这之后，是时候创建一个 ``.pot`` 文件。一个 ``.pot`` 文件包含所有的字符串，并且它是一个 ``.po`` 文件的模板，``.po`` 文件包含已经翻译的字符串。Babel 可以为你做所有的这一切。

首先你必须进入到你的应用所在的文件夹中并且创建一个映射文件夹。对于典型的 Flask 应用，这是你要的:

.. sourcecode:: ini

    [python: **.py]
    [jinja2: **/templates/**.html]
    extensions=jinja2.ext.autoescape,jinja2.ext.with_

在你的应用中把它保存成 ``babel.cfg`` 或者其它类似的东东。接着是时候运行来自 Babel 中的 `pybabel` 命令来提取你的字符串::

    $ pybabel extract -F babel.cfg -o messages.pot .

如果你使用了 :func:`lazy_gettext` 函数，你应该告诉 pybabel，这时候需要这样运行 `pybabel`::

    $ pybabel extract -F babel.cfg -k lazy_gettext -o messages.pot .

这会使用 ``babel.cfg`` 文件中的映射并且在 ``messages.pot`` 里存储生成的模板。现在可以创建第一个翻译。例如使用这个命令可以翻译成德语::

    $ pybabel init -i messages.pot -d translations -l de

``-d translations`` 告诉 pybabel 存储翻译在这个文件夹中。这是 Flask-Babel 寻找翻译的地方。可以把它放在你的模板文件夹旁边。

现在如有必要编辑 ``translations/de/LC_MESSAGES/messages.po`` 文件。如果你感到困惑的话请参阅一些 gettext 教程。

为了能用需要编译翻译，``pybabel`` 再次大显神通::

    $ pybabel compile -d translations

如果字符串变化了怎么办？像上面一样创建一个新的 ``messages.pot`` 接着让 ``pybabel`` 整合这些变化::

    $ pybabel update -i messages.pot -d translations


之后有些字符串可能会被标记成含糊不清。如果有含糊不清的字符串的时候，务必在编译之前手动地检查他们并且移除含糊不清的标志。


问题
---------------

在 Snow Leopard 上 pybabel 最有可能会以一个异常而失败。如果发生了，检查命令的输出是否是 UTF-8::

    $ echo $LC_CTYPE
    UTF-8

不幸地这是一个 OS X 问题。为了修复它，请把如下的行加入到你的 ``~/.profile`` 文件::

    export LC_CTYPE=en_US.utf-8

接着重启你的终端。

API
---

文档这一部分列出了 Flask-Babel 中每一个公开的类或者函数。

配置
`````````````

.. autoclass:: Babel
   :members:

Context 函数
`````````````````

.. autofunction:: get_translations

.. autofunction:: get_locale

.. autofunction:: get_timezone

Datetime 函数
``````````````````

.. autofunction:: to_user_timezone

.. autofunction:: to_utc

.. autofunction:: format_datetime

.. autofunction:: format_date

.. autofunction:: format_time

.. autofunction:: format_timedelta

Gettext 函数
`````````````````

.. autofunction:: gettext

.. autofunction:: ngettext

.. autofunction:: pgettext

.. autofunction:: npgettext

.. autofunction:: lazy_gettext

.. autofunction:: lazy_pgettext

低层的 API
`````````````

.. autofunction:: refresh


.. _Flask: http://flask.pocoo.org/
.. _babel: http://babel.edgewall.org/
.. _pytz: http://pytz.sourceforge.net/
.. _speaklater: http://pypi.python.org/pypi/speaklater

Flask-Testing
*************

.. module:: flask_testing

**Flask-Testing** 扩展为 Flask 提供了单元测试的工具。

安装 Flask-Testing
======================

使用 **pip** 或者 **easy_install** 安装::

    pip install Flask-Testing

或者从版本控制系统(github)中下载最新的版本::

    git clone https://github.com/jarus/flask-testing.git
    cd flask-testing
    python setup.py develop

如果你正在使用 **virtualenv**，假设你会安装 **Flask-Testing** 在运行你的 Flask 应用程序的同一个 virtualenv 上。

编写测试用例
=============

简单地继承 ``TestCase`` 的 MyTest::

    from flask.ext.testing import TestCase

    class MyTest(TestCase):

        pass


你必须定义 ``create_app`` 方法，该方法返回一个 Flask 实例::

    from flask import Flask
    from flask.ext.testing import TestCase

    class MyTest(TestCase):

        def create_app(self):

            app = Flask(__name__)
            app.config['TESTING'] = True
            return app

如果不定义 ``create_app``，``NotImplementedError`` 异常将会抛出。


使用 LiveServer 测试
-----------------------

如果你想要你的测试通过 Selenium 或者 无头浏览器(无头浏览器的意思就是无外设的意思，可以在命令行下运行的浏览器)运行，你可以使用 LiveServerTestCase::

    import urllib2
    from flask import Flask
    from flask.ext.testing import LiveServerTestCase

    class MyTest(LiveServerTestCase):

        def create_app(self):
            app = Flask(__name__)
            app.config['TESTING'] = True
            # Default port is 5000
            app.config['LIVESERVER_PORT'] = 8943
            return app

        def test_server_is_up_and_running(self):
            response = urllib2.urlopen(self.get_server_url())
            self.assertEqual(response.code, 200)

在这个例子中 ``get_server_url`` 方法将会返回 http://localhost:8943。


测试 JSON 响应
----------------------

如果你正在测试一个返回 JSON 的视图函数的话，你可以使用 ``Response`` 对象的特殊的属性 ``json`` 来测试输出::

    @app.route("/ajax/")
    def some_json():
        return jsonify(success=True)

    class TestViews(TestCase):
        def test_some_json(self):
            response = self.client.get("/ajax/")
            self.assertEquals(response.json, dict(success=True))

选择不渲染模板
-------------------------------

当测试需要处理模板渲染的时候可能是一个大问题。如果在测试中你不想要渲染模板的话可以设置 ``render_templates`` 属性::

    class TestNotRenderTemplates(TestCase):

        render_templates = False

        def test_assert_not_process_the_template(self):
            response = self.client.get("/template/")

            assert "" == response.data

尽管可以设置不想渲染模板，但是渲染模板的信号在任何时候都会发送，你也可以使用 ``assert_template_used`` 方法来检查模板是否被渲染::

    class TestNotRenderTemplates(TestCase):

        render_templates = False

        def test_assert_mytemplate_used(self):
            response = self.client.get("/template/")

            self.assert_template_used('mytemplate.html')

当渲染模板被关闭的时候，测试执行起来会更加的快速并且视图函数的逻辑将会孤立地被测试。

使用 Twill
----------------

`Twill`_ 是一个用来通过使用命令行界面浏览网页的简单的语言。

.. note::

   请注意 Twill 只支持  Python 2.x，不能在 Python 3 或者以上版本上使用。

``Flask-Testing`` 拥有一个辅助类用来创建使用 Twill 的功能测试用例::

    def test_something_with_twill(self):

        with Twill(self.app, port=3000) as t:
            t.browser.go(t.url("/"))

旧的 ``TwillTestCase`` 类已经被弃用。

测试 SQLAlchemy
------------------

这部分将会涉及使用 **Flask-Testing** 测试 `SQLAlchemy`_ 的一部分内容。这里假设你使用的是 `Flask-SQLAlchemy`_ 扩展，并且这里的例子也不是太难，可以适用于用户自己的配置。

首先，先确保数据库的 URI 是设置成开发环境而不是生产环境！其次，一个好的测试习惯就是在每一次测试执行的时候先创建表，在结束的时候删除表，这样保证干净的测试环境::

    from flask.ext.testing import TestCase

    from myapp import create_app, db

    class MyTest(TestCase):

        SQLALCHEMY_DATABASE_URI = "sqlite://"
        TESTING = True

        def create_app(self):

            # pass in test configuration
            return create_app(self)

        def setUp(self):

            db.create_all()

        def tearDown(self):

            db.session.remove()
            db.drop_all()

同样需要注意地是每一个新的 SQLAlchemy 会话在测试用例运行的时候就被创建， ``db.session.remove()`` 在每一个测试用例的结尾被调用(这是为了确保 SQLAlchemy 会话及时被删除) - 这是一种常见的 “陷阱”。

另外一个 “陷阱” 就是 Flask-SQLAlchemy 会在每一个请求结束的时候删除 SQLAlchemy 会话(session)。因此每次调用 ``client.get()`` 或者其它客户端方法的后，SQLAlchemy 会话(session)连同添加到它的任何对象都会被删除。

例如::

    class SomeTest(MyTest):

        def test_something(self):

            user = User()
            db.session.add(user)
            db.session.commit()

            # this works
            assert user in db.session

            response = self.client.get("/")

            # this raises an AssertionError
            assert user in db.session

你现在必须重新添加 "user" 实例回 SQLAlchemy 会话(session)使用 ``db.session.add(user)``，如果你想要在数据库上做进一步的操作。

同样需要注意地是在这个例子中内存数据库 SQLite 是被使用：尽管它是十分的快，但是你要是使用其它类型的数据库(例如 MySQL 或者 PostgreSQL)，可能上述代码就不适用。

你也可能想要在 ``setUp()`` 里为你的数据库增加一组实例一旦你的数据库的表已经创建。如果你想要使用数据集的话，请参看 `Fixture`_，它包含了对 SQLAlchemy 的支持。


运行测试用例
=============

使用 unittest
--------------

一开始我建议把所有的测试放在一个文件里面，这样你可以使用 :func:`unittest.main` 函数。这个函数将会发现在你的 :class:`TestCase` 类里面的所有的测试方法。请记住，所有的测试方法和类请以 ``test`` 开头（不区分大小写），这样才能被自动识别出来。

一个测试用例的文件可以看起来像这样::

    import unittest
    import flask.ext.testing

    # your test cases

    if __name__ == '__main__':
        unittest.main()

现在你可以用 ``python tests.py`` 命令执行你的测试。

使用 nose
---------

同样 `nose`_ 也与 Flask-Testing 能够很好的融合在一起。


更新历史
==========

0.4.2 (24.07.2014)
------------------

  * Improved teardown to be more graceful.
  * Add ``message`` argument to ``assertStatus`` respectively all assertion methods with fixed status like ``assert404``.

0.4.1 (27.02.2014)
------------------

This release is dedicated to every contributer who made this release possible. Thank you very much.

  * Python 3 compatibility (without twill)
  * Add ``LiveServerTestCase``
  * Use unittest2 backports if available in python 2.6
  * Install multiprocessing for python versions earlier than 2.6

0.4 (06.07.2012)
----------------

  * Use of the new introduced import way for flask extensions. Use ``import flask.ext.testing`` instead of ``import flaskext.testing``.
  * Replace all ``assert`` with ``self.assert*`` methods for better output with unittest.
  * Improved Python 2.5 support.
  * Use Flask's preferred JSON module.


API
===

.. module:: flask.ext.testing

.. autoclass:: TestCase
   :members:

.. autoclass:: Twill
   :members:

.. autoclass:: TwillTestCase
   :members:

.. _Flask: http://flask.pocoo.org
.. _Twill: http://twill.idyll.org/
.. _Fixture: http://farmdev.com/projects/fixture/index.html
.. _SQLAlchemy: http://sqlalchemy.org
.. _Flask-SQLAlchemy: http://packages.python.org/Flask-SQLAlchemy/
.. _nose: http://nose.readthedocs.org/en/latest/

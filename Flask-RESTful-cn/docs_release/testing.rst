.. _testing:

运行测试
=================

``Makefile`` 文件中照顾到搭建一个 virtualenv 为运行测试。所有你需要做的就是运行::

    $ make test

要更改用于运行测试的 Python 版本（默认是 Python 2.7），修改上面 ``Makefile`` 文件中的 ``PYTHON_MAJOR`` 和 ``PYTHON_MINOR`` 变量。

你可以在所有支持的版本上运行测试::

    $ make test-all

单个的测试可以使用如下格式的命令运行::

    nosetests <filename>:ClassName.func_name

例如::

    $ source env/bin/activate
    $ nosetests tests/test_reqparse.py:ReqParseTestCase.test_parse_choices_insensitive

另外，提交你的更改到 Github 中你的分支上，Travis 将会自动地为你的分支运行测试。

也提供了一个 Tox 配置文件，因此你可以在本地在多个 python 版本（2.6，2.7，3.3，和 3.4）上测试 ::

    $ tox

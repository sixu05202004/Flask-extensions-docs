安装
============

该部分文档涵盖了 Flask-WTF 安装。使用任何软件包的第一步即是正确安装它。


Distribute & Pip
----------------

用 `pip <http://www.pip-installer.org/>`_ 安装 Flask-WTF 是十分简单的::

    $ pip install Flask-WTF

或者，使用 `easy_install <http://pypi.python.org/pypi/setuptools>`_::

    $ easy_install Flask-WTF

但是，你真的 `不应该这样做 <https://python-packaging-user-guide.readthedocs.org/en/latest/technical.html#pip-vs-easy-install>`_。


获取源代码
------------

Flask-WTF 在 GitHub 上活跃开发，代码在 GitHub 上 `始终可用 <https://github.com/lepture/flask-wtf>`_。

你也可以克隆公开仓库::

    git clone git://github.com/lepture/flask-wtf.git

下载 `tarball <https://github.com/lepture/flask-wtf/tarball/master>`_::

    $ curl -OL https://github.com/lepture/flask-wtf/tarball/master

或者，下载 `zipball <https://github.com/lepture/flask-wtf/zipball/master>`_::

    $ curl -OL https://github.com/lepture/flask-wtf/zipball/master


当你有一份源码的副本后，你很容易地就可以把它嵌入到你的 Python 包中，或是安装到 site-packages::

    $ python setup.py install

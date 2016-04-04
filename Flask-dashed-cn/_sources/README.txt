介绍
------------

.. image:: https://secure.travis-ci.org/jeanphix/Flask-Dashed.png

Flask-Dashed 提供构建简单以及具有扩展性的管理界面的工具。 

在线演示: http://flask-dashed.jeanphi.fr/ (需要 Github 账号)。

列表视图:

.. image:: /_static/screen.png

表单视图:

.. image:: /_static/screen-edit.png



安装
------------

    pip install Flask-Dashed


最小的使用
-------------

代码::

    from flask import Flask
    from flask_dashed.admin import Admin

    app = Flask(__name__)
    admin = Admin(app)

    if __name__ == '__main__':
        app.run()


示例应用程序: http://github.com/jeanphix/flask-dashed-demo


安全地处理
------------------

保护(“武装”)所有模块端::

    from flask import session

    book_module = admin.register_module(BookModule, '/books', 'books',
        'book management')

    @book_module.secure(http_code=401)
    def login_required():
        return "user" in session

保护(“武装”)特定的模块端::

    @book_module.secure_endpoint('edit', http_code=403)
    def check_edit_credential(view):
        # I'm now signed in, may I modify the ressource?
        return session.user.can_edit_book(view.object)


模块组织
----------------

由于管理节点( node )注册到一个“树”，因此很容易地管理它们。::

    library = admin.register_node('/library', 'library', my library)
    book_module = admin.register_module(BookModule, '/books', 'books',
        'book management', parent=library)

为了满足你的需求，导航和面包屑会自动地建立。子模块的安全将会继承自父模块。


SQLALchemy 扩展
--------------------

Code::

    from flask_dashed.ext.sqlalchemy import ModelAdminModule


    class BookModule(ModelAdminModule):
        model = Book
        db_session = db.session

    book_module = admin.register_module(BookModule, '/books', 'books',
        'book management')

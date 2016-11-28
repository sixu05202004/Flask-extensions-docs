.. _intermediate:

中高级用法
==================

.. currentmodule:: flask.ext.restful

本页涉及构建一个稍微复杂的 Flask-RESTful 应用程序，该应用程序将会覆盖到一些最佳练习当你建立一个真实世界的基于 Flask-RESTful 的 API。:ref:`quickstart` 章节适用于开始你的第一个 Flask-RESTful 应用程序，因此如果你是 Flask-RESTful 的新用户，最好查阅该章节。


项目结构
-----------------

有许多不同的方式来组织你的 Flask-RESTful 应用程序，但是这里我们描述了一个在大型的应用程序中能够很好地扩展并且维持一个不错的文件组织。

最基本的思路就是把你的应用程序分为三个主要部分。路由，资源，以及任何公共的基础部分。

下面是目录结构的一个例子： ::

    myapi/
        __init__.py
        app.py          # this file contains your app and routes
        resources/
            __init__.py
            foo.py      # contains logic for /Foo
            bar.py      # contains logic for /Bar
        common/
            __init__.py
            util.py     # just some common infrastructure

common 文件夹可能只包含一组辅助函数以满足你的应用程序公共的需求。例如，它也可能包含任何自定义输入/输出类型。

在 resource 文件夹中，你只有资源对象。因此这里就是 foo.py 可能的样子： ::

    from flask.ext import restful

    class Foo(restful.Resource):
        def get(self):
            pass
        def post(self):
            pass

app.py 中的配置就像这样： ::

    from flask import Flask
    from flask.ext import restful
    from myapi.resources.foo import Foo
    from myapi.resources.bar import Bar
    from myapi.resources.baz import Baz

    app = Flask(__name__)
    api = restful.Api(app)
    
    api.add_resource(Foo, '/Foo', '/Foo/<str:id>')
    api.add_resource(Bar, '/Bar', '/Bar/<str:id>')
    api.add_resource(Baz, '/Baz', '/Baz/<str:id>') 

因为你可能编写一个特别大型或者复杂的 API，这个文件里面会有一个所有路由以及资源的复杂列表。你也可以使用这个文件来设置任何的配置值（before_request，after_request）。基本上，这个文件配置你整个 API。


完整的参数解析示例
------------------------------

在文档的其它地方，我们已经详细地介绍了如何使用 reqparse 的例子。这里我们将设置一个有多个输入参数的资源。我们将定义一个名为 “User” 的资源。 ::

    from flask.ext import restful
    from flask.ext.restful import fields, marshal_with, reqparse

    def email(email_str):
        """ return True if email_str is a valid email """
        if valid_email(email):
            return True
        else:
            raise ValidationError("{} is not a valid email")

    post_parser = reqparse.RequestParser()
    post_parser.add_argument(
        'username', dest='username',
        type=str, location='form',
        required=True, help='The user\'s username',
    )
    post_parser.add_argument(
        'email', dest='email',
        type=email, location='form',
        required=True, help='The user\'s email',
    )
    post_parser.add_argument(
        'user_priority', dest='user_priority',
        type=int, location='form',
        default=1, choices=range(5), help='The user\'s priority',
    )

    user_fields = {
        'id': fields.Integer,
        'username': fields.String,
        'email': fields.String,
        'user_priority': fields.Integer,
        'custom_greeting': fields.FormattedString('Hey there {username}!'),
        'date_created': fields.DateTime,
        'date_updated': fields.DateTime,
        'links': fields.Nested({
            'friends': fields.Url('/Users/{id}/Friends'),
            'posts': fields.Url('Users/{id}/Posts'),
        }),
    }

    class User(restful.Resource):

        @marshal_with(user_fields)
        def post(self):
            args = post_parser.parse_args()
            user = create_user(args.username, args.email, args.user_priority)
            return user

        @marshal_with(user_fields)
        def get(self, id):
            args = get_parser.parse_args()
            user = fetch_user(id)
            return user

正如你所看到的，我们创建一个 `post_parser` 专门用来处理解析 POST 请求携带的参数。让我们逐步介绍每一个定义的参数。 ::

    post_parser.add_argument(
        'username', dest='username',
        type=str, location='form',
        required=True, help='The user\'s username',
    )

`username` 字段是所有参数中最为普通的。它从 POST 数据中获取一个字符串并且把它转换成一个字符串类型。该参数是必须得（`required=True`），这就意味着如果不提供改参数，Flask-RESTful 会自动地返回一个消息是’用户名字段是必须‘的 400 错误。 ::

    post_parser.add_argument(
        'email', dest='email',
        type=email, location='form',
        required=True, help='The user\'s email',
    )

`email` 字段是一个自定义的 `email` 类型。在最前面几行中我们定义了一个 `email` 函数，它接受一个字符串，如果该字符串类型合法的话返回 True，否则会引起一个 `ValidationError` 异常，该异常明确表示 email 类型不合法。 ::

    post_parser.add_argument(
        'user_priority', dest='user_priority',
        type=int, location='form',
        default=1, choices=range(5), help='The user\'s priority',
    )

`user_priority` 类型充分利用了 `choices` 参数。这就意味着如果提供的 `user_priority` 值不落在由 `choices` 参数指定的范围内的话，Flask-RESTful 会自动地以 400 状态码以及一个描述性的错误消息响应。

下面该讨论到输入了。我们也在 `user_fields` 字典中定义了一些有趣的字段类型用来展示一些特殊的类型。 ::

    user_fields = {
        'id': fields.Integer,
        'username': fields.String,
        'email': fields.String,
        'user_priority': fields.Integer,
        'custom_greeting': fields.FormattedString('Hey there {username}!'),
        'date_created': fields.DateTime,
        'date_updated': fields.DateTime,
        'links': fields.Nested({
            'friends': fields.Url('/Users/{id}/Friends', absolute=True),
            'posts': fields.Url('Users/{id}/Posts', absolute=True),
        }),
    }

首先，存在一个 `fields.FormattedString`。 ::

    'custom_greeting': fields.FormattedString('Hey there {username}!'),

此字段主要用于篡改响应中的值到其它的值。在这种情况下，`custom_greeting` 将总是包含从 `username` 字段返回的值。

下一步，检查 `fields.Nested`。 ::

    'links': fields.Nested({
        'friends': fields.Url('/Users/{id}/Friends', absolute=True),
        'posts': fields.Url('Users/{id}/Posts', absolute=True),
    }),

此字段是用于在响应中创建子对象。在这种情况下，我们要创建一个包含相关对象 urls 的 `links` 子对象。注意这里我们是使用了 `fields.Nested`。

最后，我们使用了 `fields.Url` 字段类型。 ::

        'friends': fields.Url('/Users/{id}/Friends', absolute=True),
        'posts': fields.Url('Users/{id}/Posts', absolute=True),

它接受一个字符串作为参数，它能以我们上面提到的 `fields.FormattedString` 同样的方式被格式化。传入 `absolute=True` 确保生成的 Urls 包含主机名。

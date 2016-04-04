.. _extending:

扩展 Flask-RESTful
=======================

.. currentmodule:: flask.ext.restful

我们认识到每一个人在 REST 框架上有着不同的需求。Flask-RESTful 试图尽可能的灵活，但是有时候你可能会发现内置的功能不足够满足你的需求。Flask-RESTful 有几个不同的扩展点，这些扩展在这种情况下会有帮助。


内容协商
-------------------

开箱即用，Flask-RESTful 仅配置为支持 JSON。我们做出这个决定是为了给 API 维护者完全控制 API 格式支持，因此一年来的路上，你不必支持那些使用 API 且用 CSV 表示的人们，甚至你都不知道他们的存在。要添加其它的 mediatypes 到你的 API 中，你需要在 :class:`~Api` 对象中声明你支持的表示。 ::

    app = Flask(__name__)
    api = restful.Api(app)

    @api.representation('application/json')
    def output_json(data, code, headers=None):
        resp = make_response(json.dumps(data), code)
        resp.headers.extend(headers or {})
        return resp

这些表示函数必须返回一个 Flask :class:`~flask.Response` 对象。


自定义字段 & 输入
----------------------

一种最常见的 Flask-RESTful 附件功能就是基于你自己数据类型的数据来定义自定义的类型或者字段。

字段
~~~~~~

自定义输出字段让你无需直接修改内部对象执行自己的输出格式。所有你必须做的就是继承 :class:`~fields.Raw` 并且实现 :meth:`~fields.Raw.format` 方法::

    class AllCapsString(fields.Raw):
        def format(self, value):
            return value.upper()
    

    # example usage
    fields = {
        'name': fields.String,
        'all_caps_name': AllCapsString(attribute=name),
    }

输入
~~~~~~

对于解析参数，你可能要执行自定义验证。创建你自己的输入类型让你轻松地扩展请求解析。 ::

    def odd_number(value):
        if value % 2 == 0:
            raise ValueError("Value is not odd")

        return value

请求解析器在你想要在错误消息中引用名称的情况下将也会允许你访问参数的名称。 ::

    def odd_number(value, name):
        if value % 2 == 0:
            raise ValueError("The parameter '{}' is not odd. You gave us the value: {}".format(name, value))

        return value

你还可以将公开的参数转换为内部表示： ::

    # maps the strings to their internal integer representation
    # 'init' => 0
    # 'in-progress' => 1
    # 'completed' => 2

    def task_status(value):
        statuses = [u"init", u"in-progress", u"completed"]
        return statuses.index(value)


然后你可以在你的 RequestParser 中使用这些自定义输入类型： ::

    parser = reqparse.RequestParser()
    parser.add_argument('OddNumber', type=odd_number)
    parser.add_argument('Status', type=task_status)
    args = parser.parse_args()


响应格式
----------------

为了支持其它的表示（像 XML,CSV,HTML），你可以使用 :meth:`~Api.representation` 装饰器。你需要在你的 API 中引用它。 ::

    api = restful.Api(app)

    @api.representation('text/csv')
    def output_csv(data, code, headers=None):
        pass
        # implement csv output!

这些输出函数有三个参数，``data``，``code``，以及 ``headers``。

``data`` 是你从你的资源方法返回的对象，``code`` 是预计的 HTTP 状态码，``headers`` 是设置在响应中任意的 HTTP 头。你的输出函数应该返回一个 Flask 响应对象。 ::

    def output_json(data, code, headers=None):
        """Makes a Flask response with a JSON encoded body"""
        resp = make_response(json.dumps(data), code)
        resp.headers.extend(headers or {})

        return resp

另外一种实现这一点的就是继承 :class:`~Api` 类并且提供你自己输出函数。 ::

    class Api(restful.Api):
        def __init__(self, *args, **kwargs):
            super(Api, self).__init__(*args, **kwargs)
            self.representations = {
                'application/xml': output_xml,
                'text/html': output_html,
                'text/csv': output_csv,
                'application/json': output_json,
            }

资源方法装饰器
--------------------------

:meth:`~flask.ext.restful.Resource` 有一个叫做 method_decorators 的属性。你可以继承 Resource 并且添加你自己的装饰器，该装饰器将会被添加到资源里面所有 ``method`` 函数。举例来说，如果你想要为每一个请求建立自定义认证。 ::

    def authenticate(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if not getattr(func, 'authenticated', True):
                return func(*args, **kwargs)
            
            acct = basic_authentication()  # custom account lookup function

            if acct:
                return func(*args, **kwargs)

            restful.abort(401)
        return wrapper


    class Resource(restful.Resource):
        method_decorators = [authenticate]   # applies to all inherited resources 

由于 Flask-RESTful Resources 实际上是 Flask 视图对象，你也可以使用标准的 `flask 视图装饰器 <http://flask.pocoo.org/docs/views/#decorating-views>`_。

自定义错误处理器
---------------------

错误处理是一个很棘手的问题。你的 Flask 应用可能身兼数职，然而你要以正确的内容类型以及错误语法处理所有的 Flask-RESTful 错误。

Flask-RESTful 在 Flask-RESTful 路由上发生任何一个 400 或者 500 错误的时候调用 :meth:`~flask.ext.restful.Api.handle_error` 函数，不会干扰到其它的路由。你可能需要你的应用程序在 404 Not Found 错误上返回一个携带正确媒体类型（介质类型）的错误信息；在这种情况下，使用 :class:`~flask.ext.restful.Api` 构造函数的 `catch_all_404s` 参数。 ::

    app = Flask(__name__)
    api = flask_restful.Api(app, catch_all_404s=True)

Flask-RESTful 会处理除了自己路由上的错误还有应用程序上所有的 404 错误。

有时候你想在发生错误的时候做一些特别的东西 - 记录到文件，发送邮件，等等。使用 :meth:`~flask.got_request_exception` 方法把自定义错误处理加入到异常。 ::

    def log_exception(sender, exception, **extra):
        """ Log an exception to our logging framework """
        sender.logger.debug('Got exception during processing: %s', exception)

    from flask import got_request_exception
    got_request_exception.connect(log_exception, app)

定义自定义错误消息
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在一个请求期间遇到某些错误的时候，你可能想返回一个特定的消息以及/或者状态码。你可以告诉 Flask-RESTful 你要如何处理每一个错误/异常，因此你不必在你的 API 代码中编写 try/except 代码块。 ::

    errors = {
        'UserAlreadyExistsError': {
            'message': "A user with that username already exists.",
            'status': 409,
        },
        'ResourceDoesNotExist': {
            'message': "A resource with that ID no longer exists.",
            'status': 410,
            'extra': "Any extra information you want.",
        },
    }

包含 `'status'` 键可以设置响应的状态码。如果没有指定的话，默认是 500.

一旦你的 `errors` 字典定义，简单地把它传给 :class:`~flask.ext.restful.Api` 构造函数。 ::

    app = Flask(__name__)
    api = flask_restful.Api(app, errors=errors)

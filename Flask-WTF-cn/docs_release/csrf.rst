CSRF 保护
===============

这部分文档介绍了 CSRF 保护。

为什么需要 CSRF？
-------------------

Flask-WTF 表单保护你免受 CSRF 威胁，你不需要有任何担心。尽管如此，如果你有不包含表单的视图，那么它们仍需要保护。

例如，由 AJAX 发送的 POST 请求，然而它背后并没有表单。在 Flask-WTF 0.9.0 以前的版本你无法获得 
CSRF 令牌。这是为什么我们要实现 CSRF。

实现
--------------

.. module:: flask_wtf.csrf

为了能够让所有的视图函数受到 CSRF 保护，你需要开启 :class:`CsrfProtect` 模块::

    from flask_wtf.csrf import CsrfProtect

    CsrfProtect(app)

像任何其它的 Flask 扩展一样，你可以惰性加载它::

    from flask_wtf.csrf import CsrfProtect

    csrf = CsrfProtect()

    def create_app():
        app = Flask(__name__)
        csrf.init_app(app)

.. note::

    你需要为 CSRF 保护设置一个秘钥。通常下，同 Flask 应用的 SECRET_KEY 是一样的。

如果模板中存在表单，你不需要做任何事情。与之前一样:

.. sourcecode:: html+jinja

    <form method="post" action="/">
        {{ form.csrf_token }}
    </form>

但是如果模板中没有表单，你仍然需要一个 CSRF 令牌:

.. sourcecode:: html+jinja

    <form method="post" action="/">
        <input type="hidden" name="csrf_token" value="{{ csrf_token() }}" />
    </form>

无论何时未通过 CSRF 验证，都会返回 400 响应。你可以自定义这个错误响应::

    @csrf.error_handler
    def csrf_error(reason):
        return render_template('csrf_error.html', reason=reason), 400

我们强烈建议你对所有视图启用 CSRF 保护。但也提供了某些视图函数不需要保护的装饰器::

    @csrf.exempt
    @app.route('/foo', methods=('GET', 'POST'))
    def my_handler():
        # ...
        return 'ok'

默认情况下你也可以在所有的视图中禁用 CSRF 保护，通过设置 ``WTF_CSRF_CHECK_DEFAULT`` 为 ``False``，仅仅当你需要的时候选择调用 ``csrf.protect()``。这也能够让你在检查 CSRF 令牌前做一些预先处理::

    @app.before_request
    def check_csrf():
        if not is_oauth(request):
            csrf.protect()

AJAX
----

不需要表单，通过 AJAX 发送 POST 请求成为可能。0.9.0 版本后这个功能变成可用的。

假设你已经使用了 ``CsrfProtect(app)``，你可以通过 ``{{ csrf_token() }}`` 获取 CSRF 令牌。这个方法在每个模板中都可以使用，你并不需要担心在没有表单时如何渲染 CSRF 令牌字段。

我们推荐的方式是在 ``<meta>`` 标签中渲染 CSRF 令牌:

.. sourcecode:: html+jinja

    <meta name="csrf-token" content="{{ csrf_token() }}">

在 ``<script>`` 标签中渲染同样可行:

.. sourcecode:: html+jinja

    <script type="text/javascript">
        var csrftoken = "{{ csrf_token() }}"
    </script>

下面的例子采用了在 ``<meta>`` 标签渲染的方式， 在 ``<script>`` 中渲染会更简单，你无须担心没有相应的例子。

无论何时你发送 AJAX POST 请求，为其添加 ``X-CSRFToken`` 头:

.. sourcecode:: javascript

    var csrftoken = $('meta[name=csrf-token]').attr('content')

    $.ajaxSetup({
        beforeSend: function(xhr, settings) {
            if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
                xhr.setRequestHeader("X-CSRFToken", csrftoken)
            }
        }
    })


故障排除
---------------

当你定义你的表单的时候，如果犯了 `这个错误`_ : 从 ``wtforms`` 中导入 ``Form`` 而不是从 ``flask.ext.wtf`` 中导入，CSRF 保护的大部分功能都能工作(除了 ``form.validate_on_submit()``)，但是 CSRF 保护将会发生异常。在提交表单的时候，你将会得到 ``Bad Request``/``CSRF token missing or incorrect`` 错误。这个错误的出现就是因为你的导入错误，而不是你的配置问题。

.. _the mistake: http://stackoverflow.com/a/20577177/884640

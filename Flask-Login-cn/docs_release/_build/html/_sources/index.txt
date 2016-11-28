===========
Flask-Login
===========
.. currentmodule:: flask.ext.login

Flask-Login 为 Flask 提供了用户会话管理。它处理了日常的登入，登出并且长时间记住用户的会话。

它会:

- 在会话中存储当前活跃的用户 ID，让你能够自由地登入和登出。
- 让你限制登入(或者登出)用户可以访问的视图。
- 处理让人棘手的 “记住我” 功能。
- 帮助你保护用户会话免遭 cookie 被盗的牵连。
- 可以与以后可能使用的 Flask-Principal 或其它认证扩展集成。

但是，它不会:

- 限制你使用特定的数据库或其它存储方法。如何加载用户完全由你决定。
- 限制你使用用户名和密码，OpenIDs，或者其它的认证方法。
- 处理超越 “登入或者登出” 之外的权限。
- 处理用户注册或者账号恢复。

.. contents::
   :local:
   :backlinks: none


配置你的应用
===============
对一个使用 Flask-Login 的应用最重要的一部分就是 `LoginManager` 类。你应该在你的代码的某处为应用创建一个，像这样::

    login_manager = LoginManager()

登录管理(login manager)包含了让你的应用和 Flask-Login 协同工作的代码，比如怎样从一个 ID 加载用户，当用户需要登录的时候跳转到哪里等等。

一旦实际的应用对象创建后，你能够这样配置它来实现登录::

    login_manager.init_app(app)


它是如何工作
=============
你必须提供一个 `~LoginManager.user_loader` 回调。这个回调用于从会话中存储的用户 ID 重新加载用户对象。它应该接受一个用户的 `unicode` ID 作为参数，并且返回相应的用户对象。比如::

    @login_manager.user_loader
    def load_user(userid):
        return User.get(userid)

如果 ID 无效的话，它应该返回 `None` (**而不是抛出异常**)。(在这种情况下，ID 会被手动从会话中移除且处理会继续)

一旦用户通过验证，你可以使用 `login_user` 函数让用户登录。例如::

    @app.route("/login", methods=["GET", "POST"])
    def login():
        form = LoginForm()
        if form.validate_on_submit():
            # login and validate the user...
            login_user(user)
            flash("Logged in successfully.")
            return redirect(request.args.get("next") or url_for("index"))
        return render_template("login.html", form=form)

就这么简单。你可用使用 `current_user` 代理来访问登录的用户，在每一个模板中都可以使用 `current_user`::

    {% if current_user.is_authenticated() %}
      Hi {{ current_user.name }}!
    {% endif %}

需要用户登入 的视图可以用 `login_required` 装饰器来装饰::

    @app.route("/settings")
    @login_required
    def settings():
        pass

当用户要登出时::

    @app.route("/logout")
    @login_required
    def logout():
        logout_user()
        return redirect(somewhere)

他们会被登出，且他们会话产生的任何 cookie 都会被清理干净。

你的用户类
===============
你用来表示用户的类需要实现这些属性和方法:

`is_authenticated`
    当用户通过验证时，也即提供有效证明时返回 `True` 。（只有通过验证的用户会满足 `login_required` 的条件。）

`is_active`
    如果这是一个活动用户且通过验证，账户也已激活，未被停用，也不符合任何你 的应用拒绝一个账号的条件，返回 `True` 。不活动的账号可能不会登入（当然， 是在没被强制的情况下）。

`is_anonymous`
    如果是一个匿名用户，返回 `True` 。（真实用户应返回 `False` 。）

`get_id()`
    返回一个能唯一识别用户的，并能用于从 `~LoginManager.user_loader` 回调中加载用户的 `unicode` 。注意着 **必须** 是一个 `unicode` —— 如果 ID 原本是 一个 `int` 或其它类型，你需要把它转换为 `unicode` 。

要简便地实现用户类，你可以从 `UserMixin` 继承，它提供了对所有这些方法的默认 实现。（虽然这不是必须的。）

定制登入过程
=============================
默认情况下，当未登录的用户尝试访问一个 `login_required` 装饰的视图，Flask-Login 会闪现一条消息并且重定向到登录视图。(如果未设置登录视图，它将会以 401 错误退出。)

登录视图的名称可以设置成 `LoginManager.login_view`。例如::

    login_manager.login_view = "users.login"

默认的闪现消息是 ``Please log in to access this page.``。要自定义该信息，请设置 `LoginManager.login_message`::

    login_manager.login_message = u"Bonvolu ensaluti por uzi tio pa臐o."

要自定义消息分类的话，请设置 `LoginManager.login_message_category`::

    login_manager.login_message_category = "info"

当重定向到登入视图，它的请求字符串中会有一个 ``next`` 变量，其值为用户之前访问的页面。

如果你想要进一步自定义登入过程，请使用 `LoginManager.unauthorized_handler` 装饰函数::

    @login_manager.unauthorized_handler
    def unauthorized():
        # do stuff
        return a_response


使用授权头(Authorization header)登录
======================================

.. Caution::
   该方法将会被弃用，使用下一节的 `~LoginManager.request_loader` 来代替。

有些时候你要支持使用 `Authorization` 头的基本认证登录，比如 API 请求。为了支持通过头登录你需要提供一个 `~LoginManager.header_loader` 回调。这个回调和 `~LoginManager.user_loader` 回调作用一样，只是它接受一个 HTTP 头(Authorization)而不是用户 id。例如::

    @login_manager.header_loader
    def load_user_from_header(header_val):
        header_val = header_val.replace('Basic ', '', 1)                                
        try:                                                                        
            header_val = base64.b64decode(header_val)                                       
        except TypeError:                                                     
            pass
        return User.query.filter_by(api_key=header_val).first()

默认情况下 `Authorization` 的值传给 `~LoginManager.header_loader` 回调。你可以使用 `AUTH_HEADER_NAME` 配置来修改使用的 HTTP 头(可以不使用 `Authorization`，使用 `Token`)。


使用 Request Loader 定制登录
=================================
有时你想要不使用 cookies 情况下登录用户，比如使用 HTTP 头或者一个作为查询参数的 api 密钥。这种情况下，你应该使用 `~LoginManager.request_loader` 回调。这个回调和 `~LoginManager.user_loader` 回调作用一样，但是 `~LoginManager.user_loader` 回调只接受 Flask 请求而不是一个 user_id。

例如，为了同时支持一个 url 参数和使用 `Authorization` 头的基本用户认证的登录::

    @login_manager.request_loader
    def load_user_from_request(request):

        # first, try to login using the api_key url arg
        api_key = request.args.get('api_key')
        if api_key:
            user = User.query.filter_by(api_key=api_key).first()
            if user:
                return user

        # next, try to login using Basic Auth
        api_key = request.headers.get('Authorization')
        if api_key:
            api_key = api_key.replace('Basic ', '', 1)                                
            try:                                                                        
                api_key = base64.b64decode(api_key)                                       
            except TypeError:                                                     
                pass
            user = User.query.filter_by(api_key=api_key).first()
            if user:
                return user

        # finally, return None if both methods did not login the user
        return None
        

匿名用户
===============
默认情况下，当一个用户没有真正地登录，`current_user` 被设置成一个 `AnonymousUserMixin` 对象。它由如下的属性和方法:

- `is_active` 和 `is_authenticated` 的值为 `False`
- `is_anonymous` 的值为 `True`
- `get_id()` 返回 `None`

如果需要为匿名用户定制一些需求(比如，需要一个权限域)，你可以向 `LoginManager` 提供一个创建匿名用户的回调（类或工厂函数）::

    login_manager.anonymous_user = MyAnonymousUser


记住我
===========
"记住我"的功能很难实现。但是，Flask-Login 几乎透明地实现它 - 只要把 ``remember=True`` 传递给 `login_user`。一个 cookie 将会存储在用户计算机中，如果用户会话中没有用户 ID 的话，Flask-Login 会自动地从 cookie 中恢复用户 ID。cookie 是防纂改的，因此如果用户纂改过它(比如，使用其它的一些东西来代替用户的 ID)，它就会被拒绝，就像不存在。

该层功能是被自动实现的。但你能（且应该，如果你的应用处理任何敏感的数据）提供 额外基础工作来增强你记住的 cookie 的安全性。


可选令牌
------------------
使用用户 ID 作为记住的令牌值不一定是安全的。更安全的方法是使用用户名和密码联合的 hash 值，或类似的东西。要添加一个额外的令牌，向你的用户对象添加一个方法：

`get_auth_token()`
    返回用户的认证令牌（返回为 `unicode` ）。这个认证令牌应能唯一识别用户，且 不易通过用户的公开信息，如 UID 和名称来猜测出——同样也不应暴露这些信息。

相应地，你应该在 `LoginManager` 上设置一个 `~LoginManager.token_loader` 函数， 它接受令牌（存储在 cookie 中）作为参数并返回合适的 User 对象。

`make_secure_token` 函数用于便利创建认证令牌。它会连接所有的参数，然后用应用的密钥来 HMAC 它确保最大的密码学安全。（如果你永久地在数据库中存储用户令牌，那么 你会希望向令牌中添加随机数据来阻碍猜测。）

如果你的应用使用密码来验证用户，在认证令牌中包含密码（或你应使用的加盐值的密码 hash ）能确保若用户更改密码，他们的旧认证令牌会失效。


”新鲜的“登录(Fresh Logins)
---------------------------
当用户登入，他们的会话被标记成“新鲜的”，就是说在这个会话只中用户实际上登录过。当会话销毁用户使用“记住我”的 cookie 重新登入，会话被标记成“非新鲜的”。`login_required` 并不在意它们之间的不同，这适用于大部分页面。然而，更改某人 的个人信息这样的敏感操作应需要一个“新鲜的”的登入。（像修改密码这样的操作总是需要 密码，无论是否重登入。）

`fresh_login_required`，除了验证用户登录，也将确保他们的登录是“新鲜的”。如果不是“新鲜的”，它会把用户送到可以重输入验证条件的页面。你可以定制 `fresh_login_required` 就像定制 `login_required` 那样，通过设置 `LoginManager.refresh_view`，`~LoginManager.needs_refresh_message`，和 
`~LoginManager.needs_refresh_message_category`::

    login_manager.refresh_view = "accounts.reauthenticate"
    login_manager.needs_refresh_message = (
        u"To protect your account, please reauthenticate to access this page."
    )
    login_manager.needs_refresh_message_category = "info"

或者提供自己的回调来处理“非新鲜的”刷新::

    @login_manager.needs_refresh_handler
    def refresh():
        # do stuff
        return a_response

调用 `confirm_login` 函数可以重新标记会话为”新鲜“。


Cookie 设置
---------------
cookie 的细节可以在应用设置中定义。

=========================== ===================================================
`REMEMBER_COOKIE_NAME`      存储“记住我”信息的 cookie 名。 
                            **默认值**： remember_token
`REMEMBER_COOKIE_DURATION`  cookie 过期时间，为一个 `datetime.timedelta` 对象。
                            **默认值：** 365 天 (1 非闰阳历年)
`REMEMBER_COOKIE_DOMAIN`    如果“记住我” cookie 应跨域，在此处设置域名值
                            （即 .example.com 会允许 example 下所有子域 名）。 
                            **默认值：** None
`REMEMBER_COOKIE_PATH`      限制”记住我“ cookie 存储到某一路径下。
                            **默认值：** ``/``
=========================== ===================================================


会话保护
==================
当上述特性保护“记住我”令牌免遭 cookie 窃取时，会话 cookie 仍然是脆弱的。 Flask-Login 包含了会话保护来帮助阻止用户会话被盗用。

你可以在 `LoginManager` 上和应用配置中配置会话保护。如果它被启用，它可以在 `basic` 或 `strong` 两种模式中运行。要在 `LoginManager` 上设置它，设置 `~LoginManager.session_protection` 属性为 ``"basic"`` 或 ``"strong"``::

    login_manager.session_protection = "strong"

或者，禁用它::

    login_manager.session_protection = None

默认，它被激活为 ``"basic"`` 模式。它可以在应用配置中设定 `SESSION_PROTECTION` 为 `None` 、 ``"basic"`` 或 ``"strong"`` 来禁用。

当启用了会话保护，每个请求，它生成一个用户电脑的标识（基本上是 IP 地址和 User Agent 的 MD5 hash 值）。如果会话不包含相关的标识，则存储生成的。如果存在标识，则匹配生成的，之后请求可用。

在 `basic` 模式下或会话是永久的，如果该标识未匹配，会话会简单地被标记为非活 跃的，且任何需要活跃登入的东西会强制用户重新验证。（当然，你必须已经使用了活跃登入机制才能奏效。）

在 `strong` 模式下的非永久会话，如果该标识未匹配，整个会话（记住的令牌如果存在，则同样）被删除。


本地化
============
默认情况下，当用户需要登录，`LoginManager` 使用 ``flash`` 来显示信息。这些信息都是英文的。如果你需要本地化，设置 `LoginManager` 的 `localize_callback` 属性为一个函数，该函数在消息被发送到 ``flash`` 的时候被调用，比如，``gettext``。


API 文档
=================
这部分文档是从 Flask-Login 源码中自动生成的。


配置登录
-----------------
.. autoclass:: LoginManager
   
   .. automethod:: setup_app
   
   .. automethod:: unauthorized
   
   .. automethod:: needs_refresh
   
   .. rubric:: General Configuration
   
   .. automethod:: user_loader
   
   .. automethod:: header_loader
   
   .. automethod:: token_loader
   
   .. attribute:: anonymous_user
   
      A class or factory function that produces an anonymous user, which
      is used when no one is logged in.
   
   .. rubric:: `unauthorized` Configuration
   
   .. attribute:: login_view
   
      The name of the view to redirect to when the user needs to log in. (This
      can be an absolute URL as well, if your authentication machinery is
      external to your application.)
   
   .. attribute:: login_message
   
      The message to flash when a user is redirected to the login page.
   
   .. automethod:: unauthorized_handler
   
   .. rubric:: `needs_refresh` Configuration
   
   .. attribute:: refresh_view
   
      The name of the view to redirect to when the user needs to
      reauthenticate.
   
   .. attribute:: needs_refresh_message
   
      The message to flash when a user is redirected to the reauthentication
      page.
   
   .. automethod:: needs_refresh_handler


登录机制
----------------
.. data:: current_user

   A proxy for the current user.

.. autofunction:: login_fresh

.. autofunction:: login_user

.. autofunction:: logout_user

.. autofunction:: confirm_login


保护视图
----------------
.. autofunction:: login_required

.. autofunction:: fresh_login_required


用户对象助手
-------------------
.. autoclass:: UserMixin
   :members:

.. autoclass:: AnonymousUser
   :members:


工具
---------
.. autofunction:: login_url

.. autofunction:: make_secure_token


信号
-------
如何在你的代码中使用这些信号请参阅 `Flask documentation on signals`_。

.. data:: user_logged_in

   当一个用户登入的时候发出。除应用（信号的发送者）之外，它还传递正登入的用户 `user` 。

.. data:: user_logged_out

   当一个用户登出的时候发出。除应用（信号的发送者）之外，它还传递正登出的用户 `user` 。

.. data:: user_login_confirmed

   当用户的登入被证实，把它标记为活跃的。（它不用于常规登入的调用。） 它不接受应用以外的任何其它参数。

.. data:: user_unauthorized

   当 `LoginManager` 上的 `unauthorized` 方法被调用时发出。它不接受应用以外的任何其它参数。

.. data:: user_needs_refresh

   当 `LoginManager` 上的 `needs_refresh` 方法被调用时发出。它不接受应用以外的任何其它参数。

.. data:: session_protected

   当会话保护起作用时，且会话被标记为非活跃或删除时发出。它不接受应用以外的任何其它参数。

.. _Flask documentation on signals: http://flask.pocoo.org/docs/signals/

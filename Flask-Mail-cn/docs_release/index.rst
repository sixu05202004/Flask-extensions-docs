flask-mail
======================================

.. module:: flask-mail

在 Web 应用中的一个最基本的功能就是能够给用户发送邮件。

**Flask-Mail** 扩展提供了一个简单的接口，可以在 `Flask`_ 应用中设置 SMTP 使得可以在视图以及脚本中发送邮件信息。

链接
-----

* `原版文档 <http://packages.python.org/Flask-Mail/>`_
* `源代码 <http://github.com/mattupstate/flask-mail>`_
* :doc:`更新历史 </changelog>`

安装 Flask-Mail
---------------------

使用 **pip** 或者 **easy_install** 安装 Flask-Mail::

    pip install Flask-Mail

或者从版本控制系统（github）中下载最新的版本::

    git clone https://github.com/mattupstate/flask-mail.git
    cd flask-mail
    python setup.py install

如果你正在使用 **virtualenv**，假设你会安装 flask-mail 在运行你的 Flask 应用程序的同一个 virtualenv 上。


配置 Flask-Mail
----------------------

**Flask-Mail** 使用标准的 Flask 配置 API 进行配置。下面这些是可用的配置型(每一个将会在文档中进行解释):

* **MAIL_SERVER** : 默认为 **'localhost'**

* **MAIL_PORT** : 默认为 **25**

* **MAIL_USE_TLS** : 默认为 **False**

* **MAIL_USE_SSL** : 默认为 **False**

* **MAIL_DEBUG** : 默认为 **app.debug**

* **MAIL_USERNAME** : 默认为 **None**

* **MAIL_PASSWORD** : 默认为 **None**

* **MAIL_DEFAULT_SENDER** : 默认为 **None**

* **MAIL_MAX_EMAILS** : 默认为 **None**

* **MAIL_SUPPRESS_SEND** : 默认为 **app.testing**

* **MAIL_ASCII_ATTACHMENTS** : 默认为 **False**

另外，**Flask-Mail** 使用标准的 Flask 的 ``TESTING`` 配置项用于单元测试(下面会具体介绍)。

邮件是通过一个 ``Mail`` 实例进行管理::

    from flask import Flask
    from flask_mail import Mail

    app = Flask(__name__)
    mail = Mail(app)

在这个例子中所有的邮件将会使用传入到 ``Mail`` 实例中的应用程序的配置项进行发送。

或者你也可以在应用程序配置的时候设置你的 ``Mail`` 实例，通过使用 **init_app** 方法::

    mail = Mail()

    app = Flask(__name__)
    mail.init_app(app)

在这个例子中邮件将会使用 Flask 的 ``current_app`` 中的配置项进行发送。如果你有多个具有不用配置项的多个应用运行在同一程序的时候，这种设置方式是十分有用的，


发送邮件
----------------

为了能够发送邮件，首先需要创建一个 ``Message`` 实例::

    from flask_mail import Message

    @app.route("/")
    def index():

        msg = Message("Hello",
                      sender="from@example.com",
                      recipients=["to@example.com"])

你能够设置一个或者多个收件人::

    msg.recipients = ["you@example.com"]
    msg.add_recipient("somebodyelse@example.com")

如果你设置了 ``MAIL_DEFAULT_SENDER``，就不必再次填写发件人，默认情况下将会使用配置项的发件人::

    msg = Message("Hello",
                  recipients=["to@example.com"])

如果 ``sender`` 是一个二元组，它将会被分成姓名和邮件地址::

    msg = Message("Hello",
                  sender=("Me", "me@example.com"))

    assert msg.sender == "Me <me@example.com>"

邮件内容可以包含主体以及/或者 HTML::

    msg.body = "testing"
    msg.html = "<b>testing</b>"

最后，发送邮件的时候请使用 Flask 应用设置的 ``Mail`` 实例::

    mail.send(msg)


大量邮件
-----------

通常在一个 Web 应用中每一个请求会同时发送一封或者两封邮件。在某些特定的场景下，有可能会发送数十或者数百封邮件，不过这种发送工作会给交离线任务或者脚本执行。

在这种情况下发送邮件的代码会有些不同::

    with mail.connect() as conn:
        for user in users:
            message = '...'
            subject = "hello, %s" % user.name
            msg = Message(recipients=[user.email],
                          body=message,
                          subject=subject)

            conn.send(msg)


与电子邮件服务器的连接会一直保持活动状态直到所有的邮件都已经发送完成后才会关闭（断开）。

有些邮件服务器会限制一次连接中的发送邮件的上限。你可以设置重连前的发送邮件的最大数，通过配置 **MAIL_MAX_EMAILS** 。


附件
-----------

在邮件中添加附件同样非常简单::

    with app.open_resource("image.png") as fp:
        msg.attach("image.png", "image/png", fp.read())

具体细节请参看 `API`_ 。

如果 ``MAIL_ASCII_ATTACHMENTS`` 设置成 **True** 的话，文件名将会转换成 ASCII 的。
当文件名是以 UTF-8 编码的时候，使用邮件转发的时候会修改邮件内容并且混淆 Content-Disposition 描述，这个时候 ``MAIL_ASCII_ATTACHMENTS`` 配置项是十分有用的。转换成 ASCII 的基本方式就是对 non-ASCII 字符的去除。任何一个 unicode 字符能够被 NFKD 分解成一个或者多个 ASCII 字符。


单元测试以及禁止发送邮件
---------------------------------

当在单元测试中，或者在一个开发环境中，能够禁止邮件发送是十分有用的。

如果设置项 ``TESTING`` 设置成 ``True``，emails 将会被禁止发送。调用 ``send()`` 发送邮件实际上不会有任何邮件被发送。

另外在测试环境之中的话，你可以设置 ``MAIL_SUPPRESS_SEND`` 为 **True**，这也会有相同的效果。

然而，当单元测试的时候追踪邮件是否发送成功也是十分有用的。

为了能够追踪发送邮件的“轨迹”，可以使用 ``record_messages`` 方法::

    with mail.record_messages() as outbox:

        mail.send_message(subject='testing',
                          body='test',
                          recipients=emails)

        assert len(outbox) == 1
        assert outbox[0].subject == "testing"

**outbox** 是一个 发送 ``Message`` 实例的列表。

为了使得上述代码能够正常运行，必须安装 blinker 包。

需要注意的是以前的处理方式，即把 **outbox** 赋予给 ``g`` 对象已经过时。


头注入
----------------

为了防止 `header injection <http://www.nyphp.org/PHundamentals/8_Preventing-Email-Header-Injection>`_，尝试着在邮件主题、发件人或者收件人中使用换行符将会抛出 ``BadHeaderError`` 异常。

信号量
------------------

.. versionadded:: 0.4

**Flask-Mail** 现在通过 ``email_dispatched`` 信号开始支持信号量。只要邮件被调度，信号就会发送(即使邮件没有真正的发送，例如，在测试环境中)。

订阅 ``email_dispatched`` 信号的函数使用 ``Message`` 实例作为第一参数，Flask app 实例作为一个可选的参数::

    def log_message(message, app):
        app.logger.debug(message.subject)

    email_dispatched.connect(log_message)


API
---

.. module:: flask_mail

.. autoclass:: Mail
   :members: send, connect, send_message

.. autoclass:: Attachment

.. autoclass:: Connection
   :members: send, send_message

.. autoclass:: Message
   :members: attach, add_recipient

.. _Flask: http://flask.pocoo.org
.. _GitHub: http://github.com/mattupstate/flask-mail

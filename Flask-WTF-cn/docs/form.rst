创建表单
==============

这部分文档涉及表单(Form)信息。

安全表单
-----------

.. module:: flask_wtf

无需任何配置，:class:`Form` 是一个带有 CSRF 保护的并且会话安全的表单。我们鼓励你什么都不做。

但是如果你想要禁用 CSRF 保护，你可以这样::

    form = Form(csrf_enabled=False)

如果你想要全局禁用 CSRF 保护，你真的不应该这样做。但是你要坚持这样做的话，你可以在配置中这样写::

    WTF_CSRF_ENABLED = False

为了生成 CSRF 令牌，你必须有一个密钥，这通常与你的 Flask 应用密钥一致。如果你想使用不同的密钥，可在配置中指定::

    WTF_CSRF_SECRET_KEY = 'a random string'


文件上传
------------

.. module:: flask_wtf.file

Flask-WTF 提供 :class:`FileField` 来处理文件上传，它在表单提交后，自动从 ``flask.request.files`` 中抽取数据。:class:`FileField` 的 ``data`` 属性是一个 Werkzeug FileStorage 实例。

例如::

    from werkzeug import secure_filename
    from flask_wtf.file import FileField

    class PhotoForm(Form):
        photo = FileField('Your photo')

    @app.route('/upload/', methods=('GET', 'POST'))
    def upload():
        form = PhotoForm()
        if form.validate_on_submit():
            filename = secure_filename(form.photo.data.filename)
            form.photo.data.save('uploads/' + filename)
        else:
            filename = None
        return render_template('upload.html', form=form, filename=filename)

.. note::

    记得把你的 HTML 表单的 ``enctype`` 设置成 ``multipart/form-data``，既是:

    .. sourcecode:: html

        <form action="/upload/" method="POST" enctype="multipart/form-data">
            ....
        </form>

此外，Flask-WTF 支持文件上传的验证。提供了 :class:`FileRequired` 和 :class:`FileAllowed`。

:class:`FileAllowed` 能够很好地和 Flask-Uploads 一起协同工作, 例如::

    from flask.ext.uploads import UploadSet, IMAGES
    from flask_wtf import Form
    from flask_wtf.file import FileField, FileAllowed, FileRequired

    images = UploadSet('images', IMAGES)

    class UploadForm(Form):
        upload = FileField('image', validators=[
            FileRequired(),
            FileAllowed(images, 'Images only!')
        ])

也能在没有 Flask-Uploads 下挑大梁。这时候你需要向 :class:`FileAllowed` 传入扩展名即可::

    class UploadForm(Form):
        upload = FileField('image', validators=[
            FileRequired(),
            FileAllowed(['jpg', 'png'], 'Images only!')
        ])

HTML5 控件
-------------

.. note::

    自 wtforms 1.0.5 版本开始，wtforms 就内嵌了 HTML5 控件和字段。如果可能的话，你可以考虑从 wtforms 中导入它们。 

    我们将会在 0.9.3 版本后移除 html5 模块。


你可以从 ``wtforms`` 中导入一些 HTML5 控件::

    from wtforms.fields.html5 import URLField
    from wtforms.validators import url

    class LinkForm(Form):
        url = URLField(validators=[url()])


.. _recaptcha:

验证码
---------

.. module:: flask_wtf.recaptcha

Flask-WTF 通过 :class:`RecaptchaField` 也提供对验证码的支持::

    from flask_wtf import Form, RecaptchaField
    from wtforms import TextField

    class SignupForm(Form):
        username = TextField('Username')
        recaptcha = RecaptchaField()

这伴随着诸多配置，你需要逐一地配置它们。

===================== ===============================================
RECAPTCHA_PUBLIC_KEY  **必须** 公钥
RECAPTCHA_PRIVATE_KEY **必须** 私钥
RECAPTCHA_API_SERVER  **可选** 验证码 API 服务器
RECAPTCHA_PARAMETERS  **可选** 一个 JavaScript（api.js）参数的字典
RECAPTCHA_DATA_ATTRS  **可选** 一个数据属性项列表
                      https://developers.google.com/recaptcha/docs/display
===================== ===============================================

RECAPTCHA_PARAMETERS 和 RECAPTCHA_DATA_ATTRS 的示例::

    RECAPTCHA_PARAMETERS = {'hl': 'zh', 'render': 'explicit'}
    RECAPTCHA_DATA_ATTRS = {'theme': 'dark'}

对于应用测试时，如果 ``app.testing`` 为 ``True`` ，考虑到方便测试，Recaptcha 字段总是有效的。

在模板中很容易添加 Recaptcha 字段:

.. sourcecode:: html+jinja

    <form action="/" method="post">
        {{ form.username }}
        {{ form.recaptcha }}
    </form>

我们为你提供了例子: `recaptcha@github`_。

.. _`recaptcha@github`: https://github.com/lepture/flask-wtf/tree/master/examples/recaptcha

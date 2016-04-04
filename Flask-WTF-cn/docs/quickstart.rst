快读入门
==========

急于上手？本页对 Flask-WTF 给出了一个详尽的介绍。假设你已经安装了 Flask-WTF，如果还未安装的话，请先浏览 :doc:`install`。


创建表单
--------------

Flask-WTF 提供了对 WTForms 的集成。例如::

    from flask_wtf import Form
    from wtforms import StringField
    from wtforms.validators import DataRequired

    class MyForm(Form):
        name = StringField('name', validators=[DataRequired()])


.. note::

   从 0.9.0 版本开始，Flask-WTF 将不会从 wtforms 中导入任何的内容，用户必须自己从 wtforms 中导入字段。

另外，隐藏的 CSRF 令牌会被自动地创建。你可以在模板这样地渲染它:

.. sourcecode:: html+jinja

    <form method="POST" action="/">
        {{ form.csrf_token }}
        {{ form.name.label }} {{ form.name(size=20) }}
        <input type="submit" value="Go">
    </form>

尽管如此，为了创建有效的 XHTML/HTML， Form 类有一个 hidden_tag 方法， 它在一个隐藏的 DIV 标签中渲染任何隐藏的字段，包括 CSRF 字段:

.. sourcecode:: html+jinja

    <form method="POST" action="/">
        {{ form.hidden_tag() }}
        {{ form.name.label }} {{ form.name(size=20) }}
        <input type="submit" value="Go">
    </form>


验证表单
----------------

在你的视图处理程序中验证请求::

    @app.route('/submit', methods=('GET', 'POST'))
    def submit():
        form = MyForm()
        if form.validate_on_submit():
            return redirect('/success')
        return render_template('submit.html', form=form)

注意你不需要把 ``request.form`` 传给 Flask-WTF；Flask-WTF 会自动加载。便捷的方法 ``validate_on_submit`` 将会检查是否是一个 POST 请求并且请求是否有效。

阅读 :doc:`form` 学习更多的关于表单的技巧。

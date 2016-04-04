开发者接口
===================

该部分文档涵盖了 Flask-WTF 的全部接口。

表单和字段
----------------

.. module:: flask_wtf

.. autoclass:: Form
   :members:

.. autoclass:: RecaptchaField

.. autoclass:: Recaptcha

.. autoclass:: RecaptchaWidget

.. module:: flask_wtf.file

.. autoclass:: FileField
   :members:

.. autoclass:: FileAllowed

.. autoclass:: FileRequired

.. module:: flask_wtf.html5

.. autoclass:: SearchInput

.. autoclass:: SearchField

.. autoclass:: URLInput

.. autoclass:: URLField

.. autoclass:: EmailInput

.. autoclass:: EmailField

.. autoclass:: TelInput

.. autoclass:: TelField

.. autoclass:: NumberInput

.. autoclass:: IntegerField

.. autoclass:: DecimalField

.. autoclass:: RangeInput

.. autoclass:: IntegerRangeField

.. autoclass:: DecimalRangeField


CSRF 保护
---------------

.. module:: flask_wtf.csrf

.. autoclass:: CsrfProtect
   :members:

.. autofunction:: generate_csrf

.. autofunction:: validate_csrf

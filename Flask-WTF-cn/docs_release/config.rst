配置
=============

这里是所有配置的全表。

表单和 CSRF
--------------

Flask-WTF 完整的配置清单。通常，你不必配置它们。默认的配置就能正常工作。

======================== ===============================================
WTF_CSRF_ENABLED         禁用/开启表单的 CSRF 保护。默认是开启。
WTF_CSRF_CHECK_DEFAULT   默认下启用 CSRF 检查针对所有的视图。
                         默认值是 True。
WTF_I18N_ENABLED         禁用/开启 I18N 支持。需要和 Flask-Babel 
                         配合一起使用。默认是开启。
WTF_CSRF_HEADERS         需要检验的 CSRF 令牌 HTTP 头。默认是
                         **['X-CSRFToken', 'X-CSRF-Token']**
WTF_CSRF_SECRET_KEY      一个随机字符串生成 CSRF 令牌。
                         默认同 SECRET_KEY 一样。
WTF_CSRF_TIME_LIMIT      CSRF 令牌过期时间。默认是 **3600** 秒。
WTF_CSRF_SSL_STRICT      使用 SSL 时进行严格保护。这会检查 HTTP Referrer，
                         验证是否同源。默认为 True 。
WTF_CSRF_METHODS         使用 CSRF 保护的 HTTP 方法。默认是 
                         **['POST', 'PUT', 'PATCH']**
WTF_HIDDEN_TAG           隐藏的 HTML 标签包装的名称。
                         默认是 **div**
WTF_HIDDEN_TAG_ATTRS     隐藏的 HTML 标签包装的标签属性。
                         默认是 **{'style': 'display:none;'}**
======================== ===============================================


验证码
---------

你已经在 :ref:`recaptcha` 中了解了这些配置选项。该表为了方便速查。

===================== ==============================================
RECAPTCHA_USE_SSL     允许/禁用 Recaptcha 使用 SSL。默认是 False。
RECAPTCHA_PUBLIC_KEY  **必须** 公钥。
RECAPTCHA_PRIVATE_KEY **必须** 私钥。
RECAPTCHA_OPTIONS     **可选** 配置选项的字典。
                      https://www.google.com/recaptcha/admin/create
===================== ==============================================

.. _fields:

输出字段
===============

.. currentmodule:: flask.ext.restful

Flask-RESTful 提供了一个简单的方式来控制在你的响应中实际呈现什么数据。使用 ``fields`` 模块，你可以使用在你的资源里的任意对象（ORM 模型、定制的类等等）并且 ``fields`` 让你格式化和过滤响应，因此您不必担心暴露内部数据结构。

当查询你的代码的时候，哪些数据会被呈现以及它们如何被格式化是很清楚的。


基本用法
-----------

你可以定义一个字典或者 ``fields`` 的 OrderedDict 类型，OrderedDict 类型是指键名是要呈现的对象的属性或键的名称，键值是一个类，该类格式化和返回的该字段的值。这个例子有三个字段，两个是字符串（Strings）以及一个是日期时间（DateTime），格式为 RFC 822 日期字符串（同样也支持 ISO 8601） ::

    from flask.ext.restful import Resource, fields, marshal_with

    resource_fields = {
        'name': fields.String,
        'address': fields.String,
        'date_updated': fields.DateTime(dt_format='rfc822'),
    }
    
    class Todo(Resource):
        @marshal_with(resource_fields, envelope='resource')
        def get(self, **kwargs):
            return db_get_todo()  # Some function that queries the db

这个例子假设你有一个自定义的数据库对象（``todo``），它具有属性：``name``， ``address``， 以及 ``date_updated``。该对象上任何其它的属性可以被认为是私有的不会在输出中呈现出来。一个可选的 ``envelope`` 关键字参数被指定为封装结果输出。

装饰器 ``marshal_with`` 是真正接受你的数据对象并且过滤字段。``marshal_with`` 能够在单个对象，字典，或者列表对象上工作。

注意：marshal_with 是一个很便捷的装饰器，在功能上等效于如下的 ``return marshal(db_get_todo(), resource_fields), 200``。这个明确的表达式能用于返回 200 以及其它的 HTTP 状态码作为成功响应（错误响应见 ``abort``）。


重命名属性
-------------------

很多时候你面向公众的字段名称是不同于内部的属性名。使用 ``attribute`` 可以配置这种映射。 ::

    fields = {
        'name': fields.String(attribute='private_name'),
        'address': fields.String,
    }

lambda 也能在 ``attribute`` 中使用 ::

    fields = {
        'name': fields.String(attribute=lambda x: x._private_name),
        'address': fields.String,
    }


默认值
--------------

如果由于某种原因你的数据对象中并没有你定义的字段列表中的属性，你可以指定一个默认值而不是返回 ``None``。 ::

    fields = {
        'name': fields.String(default='Anonymous User'),
        'address': fields.String,
    }


自定义字段&多个值
-------------------------------

有时候你有你自己定义格式的需求。你可以继承 ``fields.Raw`` 类并且实现格式化函数。当一个属性存储多条信息的时候是特别有用的。例如，一个位域（bit-field）各位代表不同的值。你可以使用 ``fields`` 复用一个单一的属性到多个输出值（一个属性在不同情况下输出不同的结果）。

这个例子假设在 ``flags`` 属性的第一位标志着一个“正常”或者“迫切”项，第二位标志着“读”与“未读”。这些项可能很容易存储在一个位字段，但是可读性不高。转换它们使得具有良好的可读性是很容易的。 ::

    class UrgentItem(fields.Raw):
        def format(self, value):
            return "Urgent" if value & 0x01 else "Normal"

    class UnreadItem(fields.Raw):
        def format(self, value):
            return "Unread" if value & 0x02 else "Read" 

    fields = {
        'name': fields.String,
        'priority': UrgentItem(attribute='flags'),
        'status': UnreadItem(attribute='flags'),
    }

Url & 其它具体字段
---------------------------

Flask-RESTful 包含一个特别的字段，``fields.Url``，即为所请求的资源合成一个 uri。这也是一个好示例，它展示了如何添加并不真正在你的数据对象中存在的数据到你的响应中。 ::

    class RandomNumber(fields.Raw):
        def output(self, key, obj):
            return random.random()

    fields = {
        'name': fields.String,
        # todo_resource is the endpoint name when you called api.add_resource() 
        'uri': fields.Url('todo_resource'),
        'random': RandomNumber,
    }


默认情况下，``fields.Url`` 返回一个相对的 uri。为了生成包含协议（scheme），主机名以及端口的绝对 uri，需要在字段声明的时候传入 ``absolute=True``。传入 ``scheme`` 关键字参数可以覆盖默认的协议（scheme）::

    fields = {
        'uri': fields.Url('todo_resource', absolute=True)
        'https_uri': fields.Url('todo_resource', absolute=True, scheme='https')
    }

复杂结构
------------------

你可以有一个扁平的结构，marshal_with 将会把它转变为一个嵌套结构 ::

    >>> from flask.ext.restful import fields, marshal
    >>> import json
    >>>
    >>> resource_fields = {'name': fields.String}
    >>> resource_fields['address'] = {}
    >>> resource_fields['address']['line 1'] = fields.String(attribute='addr1')
    >>> resource_fields['address']['line 2'] = fields.String(attribute='addr2')
    >>> resource_fields['address']['city'] = fields.String
    >>> resource_fields['address']['state'] = fields.String
    >>> resource_fields['address']['zip'] = fields.String
    >>> data = {'name': 'bob', 'addr1': '123 fake street', 'addr2': '', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
    >>> json.dumps(marshal(data, resource_fields))
    '{"name": "bob", "address": {"line 1": "123 fake street", "line 2": "", "state": "NY", "zip": "10468", "city": "New York"}}'

注意：address 字段并不真正地存在于数据对象中，但是任何一个子字段（sub-fields）可以直接地访问对象的属性，就像没有嵌套一样。

.. _list-field:

列表字段
----------

你也可以把字段解组（unmarshal）成列表 ::

    >>> from flask.ext.restful import fields, marshal
    >>> import json
    >>> 
    >>> resource_fields = {'name': fields.String, 'first_names': fields.List(fields.String)}
    >>> data = {'name': 'Bougnazal', 'first_names' : ['Emile', 'Raoul']}
    >>> json.dumps(marshal(data, resource_fields))
    >>> '{"first_names": ["Emile", "Raoul"], "name": "Bougnazal"}'

.. _nested-field:

高级：嵌套字段
-----------------------

尽管使用字典套入字段能够使得一个扁平的数据对象变成一个嵌套的响应，你可以使用 ``Nested`` 解组（unmarshal）嵌套数据结构并且合适地呈现它们。 ::

    >>> from flask.ext.restful import fields, marshal
    >>> import json
    >>>   
    >>> address_fields = {}
    >>> address_fields['line 1'] = fields.String(attribute='addr1')
    >>> address_fields['line 2'] = fields.String(attribute='addr2')
    >>> address_fields['city'] = fields.String(attribute='city')
    >>> address_fields['state'] = fields.String(attribute='state')
    >>> address_fields['zip'] = fields.String(attribute='zip')
    >>> 
    >>> resource_fields = {}
    >>> resource_fields['name'] = fields.String
    >>> resource_fields['billing_address'] = fields.Nested(address_fields)
    >>> resource_fields['shipping_address'] = fields.Nested(address_fields)
    >>> address1 = {'addr1': '123 fake street', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
    >>> address2 = {'addr1': '555 nowhere', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
    >>> data = { 'name': 'bob', 'billing_address': address1, 'shipping_address': address2}
    >>> 
    >>> json.dumps(marshal_with(data, resource_fields))
    '{"billing_address": {"line 1": "123 fake street", "line 2": null, "state": "NY", "zip": "10468", "city": "New York"}, "name": "bob", "shipping_address": {"line 1": "555 nowhere", "line 2": null, "state": "NY", "zip": "10468", "city": "New York"}}'

此示例使用两个嵌套字段。``Nested`` 构造函数把字段的字典作为子字段（sub-fields）来呈现。使用 ``Nested`` 和之前例子中的嵌套字典之间的重要区别就是属性的上下文。在本例中 "billing_address" 是一个具有自己字段的复杂的对象，传递给嵌套字段的上下文是子对象（sub-object），而不是原来的“数据”对象。换句话说，``data.billing_address.addr1`` 是在这里的范围（译者：这里是直译），然而在之前例子中的 ``data.addr1`` 是位置属性。记住：嵌套和列表对象创建一个新属性的范围。

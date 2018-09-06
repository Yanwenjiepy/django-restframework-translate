source: settings.py

# Settings

> Namespaces are one honking great idea - let's do more of those!
>
> 命名空间是一个非常棒的主意，让我们用它来一起搞事情吧！
>
> &mdash; [The Zen of Python][cite]

Configuration for REST framework is all namespaced inside a single Django setting, named `REST_FRAMEWORK`.

`REST framework`的所有相关配置都在`settings.py`中的`REST_FRAMEWORK`下进行配置，那么这个`RESR_FRAMEWORK`就是`REST framework`的命名空间。

For example your project's `settings.py` file might include something like this:

你打开你现有项目（使用了`REST Framework`框架）中的`settings.py`文件，找到`REST_FRAMEWORK`相关的信息，他应该是类似下面这样的：

```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
    ),
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    )
}
```



## Accessing settings

If you need to access the values of REST framework's API settings in your project,
you should use the `api_settings` object.  For example.

如果你想在你的项目中查看`REST framework‘s API`的相关配置信息，你可以使用`api_settings`object来查看，就像下面这样使用：

```python
from rest_framework.settings import api_settings

print api_settings.DEFAULT_AUTHENTICATION_CLASSES
```

The `api_settings` object will check for any user-defined settings, and otherwise fall back to the default values.  Any setting that uses string import paths to refer to a class will automatically import and return the referenced class, instead of the string literal.

`api_settings`onject 将会检查程序员是否对相关设置进行了定义，如果程序员没有对这些设置进行定义，那么将会返回默认值。如果你在设置中导入某个路径下的Class，那么`api_settings`object 将只返回这个Class，而不会包含你在配置时添加的路径信息。

---

# API Reference

## API policy settings

API基本设置

*The following settings control the basic API policies, and are applied to every `APIView` class-based view, or `@api_view` function based view.*

*下面这些设置是基本的API设置，这些设置对每个`APIView`和`@api_view`都有效*

#### DEFAULT_RENDERER_CLASSES

A list or tuple of renderer classes, that determines the default set of renderers that may be used when returning a `Response` object.

项目使用的`renderer` classes 的集合，可以是一个tuple或list。根据它们来完成对`Response`object 的渲染。

Default:

```python
(
    'rest_framework.renderers.JSONRenderer',
    'rest_framework.renderers.BrowsableAPIRenderer',
)
```



#### DEFAULT_PARSER_CLASSES

A list or tuple of parser classes, that determines the default set of parsers used when accessing the `request.data` property.

项目使用的`parser`classes 的集合，可以是一个tuple或list。根据他们来完成对`request.data`的解析。

Default:

```python
(
    'rest_framework.parsers.JSONParser',
    'rest_framework.parsers.FormParser',
    'rest_framework.parsers.MultiPartParser'
)
```



#### DEFAULT_AUTHENTICATION_CLASSES

A list or tuple of authentication classes, that determines the default set of authenticators used when accessing the `request.user` or `request.auth` properties.

项目使用的`authentication`classes的集合，可以是一个tuple或list。通过他们访问`request.user`和`request.auth`属性来判断发出request的用户的具体身份。

Default:

```python
(
    'rest_framework.authentication.SessionAuthentication',
    'rest_framework.authentication.BasicAuthentication'
)
```



#### DEFAULT_PERMISSION_CLASSES

A list or tuple of permission classes, that determines the default set of permissions checked at the start of a view. Permission must be granted by every class in the list.

项目所使用的`permission`class 的集合，可以是一个tuple或list。在执行所有的view之前，将对request进行权限检查，只有符合list中所有权限要求的情况下才能够执行view。

Default:

```python
(
    'rest_framework.permissions.AllowAny',
)
```



#### DEFAULT_THROTTLE_CLASSES

A list or tuple of throttle classes, that determines the default set of throttles checked at the start of a view.

项目所使用的throttle classes 列表或者元组。在开始执行所有的view之前，需要检查request是否符合throttle的要求，不符合要求的request将被拒绝执行view。

Default: `()`



#### DEFAULT_CONTENT_NEGOTIATION_CLASS

A content negotiation class, that determines how a renderer is selected for the response, given an incoming request.

默认的content negotiation class（内容协商类），它将根据传入的request，来为response 选择合适的renderer。

Default: `'rest_framework.negotiation.DefaultContentNegotiation'`



#### DEFAULT_SCHEMA_CLASS

A view inspector class that will be used for schema generation.

生成schema时所使用的view inspector class。

Default: `'rest_framework.schemas.AutoSchema'`



---

## Generic view settings

通用的view设置

*The following settings control the behavior of the generic class-based views.*

*下列设置用来控制 class-based views(基于类的视图)*



#### DEFAULT_PAGINATION_SERIALIZER_CLASS

---

**This setting has been removed.**

The pagination API does not use serializers to determine the output format, and
you'll need to instead override the `get_paginated_response` method on a
pagination class in order to specify how the output format is controlled.

**注意：这个设置已经被移除了**

分页API不再使用 serializer 来确定显示格式，现在你需要重写pagination class（分页类）的`get_paginated_response`方法来控制显示格式。



---

#### DEFAULT_FILTER_BACKENDS

A list of filter backend classes that should be used for generic filtering.
If set to `None` then generic filtering is disabled.

一个 filter backend classes （过滤器类）的list，用来执行通用的过滤操作。如果它被设置为`None`，过滤功能将被关闭。



#### PAGINATE_BY

---

**This setting has been removed.**

See the pagination documentation for further guidance on [setting the pagination style](pagination.md#modifying-the-pagination-style).

**注意：该设置已经被移除**

有关pagination的相关配置请参考 [setting the pagination style](pagination.md#modifying-the-pagination-style)。



---

#### PAGE_SIZE

The default page size to use for pagination.  If set to `None`, pagination is disabled by default.

Default: `None`

使用pagination时的page的大小，如果设置为`None`，pagination功能默认将被关闭。



#### PAGINATE_BY_PARAM

---

**This setting has been removed.**

See the pagination documentation for further guidance on [setting the pagination style](pagination.md#modifying-the-pagination-style).

**注意：该配置已经被移除**

更多关于分页相关的内容，请参考 [setting the pagination style](pagination.md#modifying-the-pagination-style)。



#### MAX_PAGINATE_BY

---

**This setting is pending deprecation.**

See the pagination documentation for further guidance on [setting the pagination style](pagination.md#modifying-the-pagination-style).

**注意：该配置已经被移除**

更多关于分页相关的内容，请参考 [setting the pagination style](pagination.md#modifying-the-pagination-style)。



---

### SEARCH_PARAM

The name of a query parameter, which can be used to specify the search term used by `SearchFilter`.

Default: `search`

查询参数的名称，可用于指定`SearchFilter`使用的搜索词。



#### ORDERING_PARAM

The name of a query parameter, which can be used to specify the ordering of results returned by `OrderingFilter`.

Default: `ordering`

查询参数的名称，可用于指定`OrderingFilter`返回结果的顺序。



---



## Versioning settings

版本控制设置

#### DEFAULT_VERSION

The value that should be used for `request.version` when no versioning information is present.

Default: `None`

在启用版本控制功能的情况下，当传入的请求没有版本信息时，`request.version`将默认使用该值。



#### ALLOWED_VERSIONS

If set, this value will restrict the set of versions that may be returned by the versioning scheme, and will raise an error if the provided version if not in this set.

Default: `None`

在启用该功能的情况下（将其设置为允许的 version所组成的集合），如果提供的版本不在你设置的versions set中，那么将会抛出错误。只有满足版本要求的请求才会被接受。



#### VERSION_PARAM

The string that should used for any versioning parameters, such as in the media type or URL query parameters.

Default: `'version'`

在request的url或者headers中代表版本信息的参数名字。



---



## Authentication settings

*The following settings control the behavior of unauthenticated requests.*

*下列配置对没有通过身份验证的请求有效*



#### UNAUTHENTICATED_USER

The class that should be used to initialize `request.user` for unauthenticated requests.

Default: `django.contrib.auth.models.AnonymousUser`

你需要配置一个为未通过身份验证的请求初始化`request.user`的类。



#### UNAUTHENTICATED_TOKEN

The class that should be used to initialize `request.auth` for unauthenticated requests.

Default: `None`

你需要配置一个为未通过身份验证的请求初始化`request.auth`的类。



---



## Test settings

测试配置

*The following settings control the behavior of APIRequestFactory and APIClient*

*下列配置用来控制`APIRequestFactory`和`APIClient`*



#### TEST_REQUEST_DEFAULT_FORMAT

The default format that should be used when making test requests.

This should match up with the format of one of the renderer classes in the `TEST_REQUEST_RENDERER_CLASSES` setting.

Default: `'multipart'`

在进行测试时，所选择的渲染格式。该格式应该与`TEST_REQUEST_RENDERER_CLASSES`配置的renderder的渲染格式保持一致。



#### TEST_REQUEST_RENDERER_CLASSES

The renderer classes that are supported when building test requests.

The format of any of these renderer classes may be used when constructing a test request, for example: `client.post('/users', {'username': 'jamie'}, format='json')`

在构建测试请求时，可以使用的 renderer classes（渲染器类）。你可以选择其中的任何一种来构建你的测试请求，例如这个使用`JSONRenderer`来构建的POST请求：`client.post('/users', {'username': 'jamie'}, format='json')`。

Default:

```python
(
    'rest_framework.renderers.MultiPartRenderer',
    'rest_framework.renderers.JSONRenderer'
)
```



---



## Schema generation controls

Schema配置



#### SCHEMA_COERCE_PATH_PK

If set, this maps the `'pk'` identifier in the URL conf onto the actual field
name when generating a schema path parameter. Typically this will be `'id'`.
This gives a more suitable representation as "primary key" is an implementation
detail, whereas "identifier" is a more general concept.

Default: `True`

如果设置为True，在生成schema路径参数时，`pk`标识符将被映射到数据库对应的主键字段（通常是`id`）。

`primary key`是表结构的实现细节，而`pk`标识符是一个更通用的表示。



#### SCHEMA_COERCE_METHOD_NAMES

If set, this is used to map internal viewset method names onto external action
names used in the schema generation. This allows us to generate names that
are more suitable for an external representation than those that are used
internally in the codebase.

Default: `{'retrieve': 'read', 'destroy': 'delete'}`



如果设置了Key-Value参数，则用于将内部视图集方法名称(key)映射到Schema生成中使用的外部操作名称(value)。

这样可以生成更适合外部表示的名称，而不是使用代码内部的方法名称。



---



## Content type controls



#### URL_FORMAT_OVERRIDE

The name of a URL parameter that may be used to override the default content negotiation `Accept` header behavior, by using a `format=…` query parameter in the request URL.

For example: `http://example.com/organizations/?format=csv`

If the value of this setting is `None` then URL format overrides will be disabled.

Default: `'format'`

url查询参数的名字，你可以通过该参数来查询你所期望格式的内容，在url的查询参数中使用`format=value`来设置内容的格式（format就是给配置所设置的值，默认为format，value就是格式名字）。

就像这样使用：`http://example.com/organizations/?format=csv`

如果你将其设置为`None`，那么该功能将被禁用。



#### FORMAT_SUFFIX_KWARG

The name of a parameter in the URL conf that may be used to provide a format suffix. This setting is applied when using `format_suffix_patterns` to include suffixed URL patterns.

For example: `http://example.com/organizations.csv/`

Default: `'format'`

URL conf中提供格式后缀的参数名字。

使用`format_suffix_patterns`后缀URL模式时，将应用此设置。

此时你需要将该配置作为一个关键字参数添加到你的view中。



在绝大多数情况下，以上俩个配置使用默认配置即可。

---



## Date and time formatting

*The following settings are used to control how date and time representations may be parsed and rendered.*

*以下配置与日期和时间的解析与呈现相关*



#### DATETIME_FORMAT

A format string that should be used by default for rendering the output of `DateTimeField` serializer fields.  If `None`, then `DateTimeField` serializer fields will return Python `datetime` objects, and the datetime encoding will be determined by the renderer.

May be any of `None`, `'iso-8601'` or a Python [strftime format][strftime] string.

Default: `'iso-8601'`

格式化字符串，用于渲染`DateTimeField`serializer 返回的内容。

如果将该值设置为`None`，`DateTimeField` serializer 将返回一个Python `datatime` object，其编码将由renderer来决定。

该配置可以是`None`、`‘iso-8601’`、Python strftime format 字符串中的任意一个。



#### DATETIME_INPUT_FORMATS

A list of format strings that should be used by default for parsing inputs to `DateTimeField` serializer fields.

May be a list including the string `'iso-8601'` or Python [strftime format][strftime] strings.

Default: `['iso-8601']`

格式化字符串的列表，用于解析`DataTimeField`serializer 返回的内容。

该列表中包括`‘iso-8601’`、或Python strftime format 字符串。



#### DATE_FORMAT

A format string that should be used by default for rendering the output of `DateField` serializer fields.  If `None`, then `DateField` serializer fields will return Python `date` objects, and the date encoding will be determined by the renderer.

May be any of `None`, `'iso-8601'` or a Python [strftime format][strftime] string.

Default: `'iso-8601'`

格式化字符串，用于渲染`DateField`serializer fields返回的内容 。

如果将该值设置为`None`，`DateField` serializer 将返回一个Python `data` object，其编码将由renderer来决定。

该配置可以是`None`、`‘iso-8601’`、Python strftime format 字符串中的任意一个。



#### DATE_INPUT_FORMATS

A list of format strings that should be used by default for parsing inputs to `DateField` serializer fields.

May be a list including the string `'iso-8601'` or Python [strftime format][strftime] strings.

Default: `['iso-8601']`

格式化字符串的列表，用于解析`DataField`serializer 返回的内容。

该列表中包括`‘iso-8601’`、或Python strftime format 字符串。



#### TIME_FORMAT

A format string that should be used by default for rendering the output of `TimeField` serializer fields.  If `None`, then `TimeField` serializer fields will return Python `time` objects, and the time encoding will be determined by the renderer.

May be any of `None`, `'iso-8601'` or a Python [strftime format][strftime] string.

Default: `'iso-8601'`

格式化字符串，用于渲染`TimeField`serializer 返回的内容。

如果将该值设置为`None`，`TimeField` serializer 将返回一个Python `time` object，其编码将由renderer来决定。

该配置可以是`None`、`‘iso-8601’`、Python strftime format 字符串中的任意一个。



#### TIME_INPUT_FORMATS

A list of format strings that should be used by default for parsing inputs to `TimeField` serializer fields.

May be a list including the string `'iso-8601'` or Python [strftime format][strftime] strings.

Default: `['iso-8601']`

一个格式字符串的列表。用于解析`TimeField`serializer 返回的内容。

该列表中包括`‘iso-8601’`、或Python strftime format 字符串。



---



## Encodings



#### UNICODE_JSON

When set to `True`, JSON responses will allow unicode characters in responses. For example:

    {"unicode black star":"★"}

When set to `False`, JSON responses will escape non-ascii characters, like so:

    {"unicode black star":"\u2605"}

Both styles conform to [RFC 4627][rfc4627], and are syntactically valid JSON. The unicode style is preferred as being more user-friendly when inspecting API responses.

Default: `True`

如果将UNICODE_JSON设置为`True`，那么JSON response 将允许unicode字符存在；

如果将UNICODE_JSON设置为`False`，那么JSON response 将只允许ASCII字符存在，非ASCII字符将被转义为ASCII编码。

这两种样式都符合RFC 4627，并且都符合JSON语法。

但是在检查API响应时，首选unicode样式，因为它对用户更加友好。



#### COMPACT_JSON

When set to `True`, JSON responses will return compact representations, with no spacing after `':'` and `','` characters. For example:

    {"is_admin":false,"email":"jane@example"}

When set to `False`, JSON responses will return slightly more verbose representations, like so:

    {"is_admin": false, "email": "jane@example"}

The default style is to return minified responses, in line with [Heroku's API design guidelines][heroku-minified-json].

Default: `True`

当设置为`True`时，将返回紧凑型的结果；

当设置为`Flase`时，将返回更宽松的结果。

默认返回紧凑型的结果。



#### STRICT_JSON

When set to `True`, JSON rendering and parsing will only observe syntactically valid JSON, raising an exception for the extended float values (`nan`, `inf`, `-inf`) accepted by Python's `json` module. This is the recommended setting, as these values are not generally supported. e.g., neither Javascript's `JSON.Parse` nor PostgreSQL's JSON data type accept these values.

When set to `False`, JSON rendering and parsing will be permissive. However, these values are still invalid and will need to be specially handled in your code.

Default: `True`

当设置为`True`时，将只解析和渲染标准的json格式数据，而Python的json模块中的 float values 扩展功能将会引发异常；

如果设置为`Flase`，将不会引发异常，但是你应该对这些问题进行一些处理。

因为标准的json语法并不支持 float values 的扩展功能，例如Javascript中的JSON.Parse 和PostgreSQL 中的JSON 数据类型都不支持该扩展功能，所以我们遵循JSON标准语法，默认将他设置为`True`。



#### COERCE_DECIMAL_TO_STRING

When returning decimal objects in API representations that do not support a native decimal type, it is normally best to return the value as a string. This avoids the loss of precision that occurs with binary floating point implementations.

When set to `True`, the serializer `DecimalField` class will return strings instead of `Decimal` objects. When set to `False`, serializers will return `Decimal` objects, which the default JSON encoder will return as floats.

Default: `True`

在不支持本机十进制类型的API表示形式中返回十进制对象时，通常最好将该值作为字符串返回。 

这避免了二进制浮点实现时出现的精度损失。  

设置为True时，DecimalField的serializer将返回字符串而不是Decimal对象。 

设置为False时，serializer将返回Decimal对象，默认JSON编码器将返回为浮点数。



---



## View names and descriptions

*The following settings are used to generate the view names and descriptions, as used in responses to `OPTIONS` requests, and as used in the browsable API.*

*以下设置用于生成视图名称和描述，如对OPTIONS请求的响应，以及可浏览API中使用的设置。*



#### VIEW_NAME_FUNCTION

A string representing the function that should be used when generating view names.

This should be a function with the following signature:

```python
view_name(cls, suffix=None)
```

* `cls`: The view class.  Typically the name function would inspect the name of the class when generating a descriptive name, by accessing `cls.__name__`.
* `suffix`: The optional suffix used when differentiating individual views in a viewset.

Default: `'rest_framework.views.get_view_name'`

处理views 名字生成的函数字符串表示。

- `cls`：view class。name 函数会访问view class 的`__name__`属性来获取名字
- `suffix`: 区分视图集中各个视图时使用的可选后缀。



#### VIEW_DESCRIPTION_FUNCTION

A string representing the function that should be used when generating view descriptions.

This setting can be changed to support markup styles other than the default markdown.  For example, you can use it to support `rst` markup in your view docstrings being output in the browsable API.

This should be a function with the following signature:

```python
view_description(cls, html=False)
```

* `cls`: The view class.  Typically the description function would inspect the docstring of the class when generating a description, by accessing `cls.__doc__`
* `html`: A boolean indicating if HTML output is required.  `True` when used in the browsable API, and `False` when used in generating `OPTIONS` responses.

Default: `'rest_framework.views.get_view_description'`

用于处理view descriptions 的函数字符串表示。

如果你不喜欢默认的markdown，你可以自定义你喜欢的标记语言和风格。例如你可以使用`rst`在浏览器中展示view descriptions。

- `cls`: The view class.  description 函数将会访问view class的`__doc__`属性来获取描述信息。
- `html`: 一个布尔值，指示是否需要HTML输出。
- 在可浏览API中使用时将它的值设置为`True`，在用于生成OPTIONS响应时设置为`False`。



## HTML Select Field cutoffs

Global settings for [select field cutoffs for rendering relational fields](relations.md#select-field-cutoffs) in the browsable API.

在browsable API中 relational fields 的呈现数量的相关配置。



#### HTML_SELECT_CUTOFF

Global setting for the `html_cutoff` value.  Must be an integer.

Default: 1000

`html_cutoff`的值，必须是整型。



#### HTML_SELECT_CUTOFF_TEXT

A string representing a global setting for `html_cutoff_text`.

Default: `"More than {count} items..."`

`html_cutoff_text`的值，字符串格式。



---

## Miscellaneous settings

其他配置



#### EXCEPTION_HANDLER

A string representing the function that should be used when returning a response for any given exception.  If the function returns `None`, a 500 error will be raised.

This setting can be changed to support error responses other than the default `{"detail": "Failure..."}` responses.  For example, you can use it to provide API responses like `{"errors": [{"message": "Failure...", "code": ""} ...]}`.

返回任何给定异常的响应时所使用的函数。 如果函数返回`None`，则会引发500错误。

可以更改此设置以支持除默认`{“detail”：“Failure...”}`响应之外的错误响应。 

例如，您可以使用它来提供API响应，例如`{“errors”：[{“message”：“Failure ...”，“code”：“”} ...]}`。



This should be a function with the following signature:

```python
exception_handler(exc, context)
```

* `exc`: The exception.
* `exc`：给定的异常

Default: `'rest_framework.views.exception_handler'`



#### NON_FIELD_ERRORS_KEY

A string representing the key that should be used for serializer errors that do not refer to a specific field, but are instead general errors.

Default: `'non_field_errors'`

应该用于序列化程序错误的 key，它并不针对于特定字段，而适用于一般错误。



#### URL_FIELD_NAME

A string representing the key that should be used for the URL fields generated by `HyperlinkedModelSerializer`.

Default: `'url'`

应该用于`HyperlinkedModelSerializer`生成的URL字段的 key。



#### NUM_PROXIES

An integer of 0 or more, that may be used to specify the number of application proxies that the API runs behind.  This allows throttling to more accurately identify client IP addresses.  If set to `None` then less strict IP matching will be used by the throttle classes.

Default: `None`

用于指定API后面运行的应用程序代理的数量，必须为整型。

这样能更准确地识别客户端IP地址。 

如果设置为`None`，则 Throttle classes 将使用不太严格的IP匹配机制。



[cite]: https://www.python.org/dev/peps/pep-0020/
[rfc4627]: http://www.ietf.org/rfc/rfc4627.txt
[heroku-minified-json]: https://github.com/interagent/http-api-design#keep-json-minified-in-all-responses
[strftime]: https://docs.python.org/3/library/time.html#time.strftime

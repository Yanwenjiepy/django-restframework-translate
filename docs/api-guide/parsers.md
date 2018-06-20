source: parsers.py

# Parsers

> Machine interacting web services tend to use more
> structured formats for sending data than form-encoded, since they're
> sending more complex data than simple forms
>
> 与使用表单数据相比，机器交互式的Web服务更加倾向于使用结构化的数据，这样能够发送更加复杂的数据。
>
> &mdash; Malcom Tredinnick, [Django developers group][cite]

REST framework includes a number of built in Parser classes, that allow you to accept requests with various media types.  There is also support for defining your own custom parsers, which gives you the flexibility to design the media types that your API accepts.

REST framework 包含很多内置的Parser classes，你可以用他们来解析各种媒体格式的请求。并且你也可以自定义Parser，让你的API更加灵活地处理请求。



## How the parser is determined

The set of valid parsers for a view is always defined as a list of classes.  When  `request.data` is accessed, REST framework will examine the `Content-Type` header on the incoming request, and determine which parser to use to parse the request content.

view可用的所有Parsers classes都在一个列表中。当`request.data`传入时，REST framework将根据`Content-Type`来选择合适的Parser来解析请求内容。

---

**Note**: When developing client applications always remember to make sure you're setting the `Content-Type` header when sending data in an HTTP request.

If you don't set the content type, most clients will default to using `'application/x-www-form-urlencoded'`, which may not be what you wanted.

As an example, if you are sending `json` encoded data using jQuery with the [.ajax() method][jquery-ajax], you should make sure to include the `contentType: 'application/json'` setting.

**注意**：在开发客户端程序时，一定要在HTTP请求中设置`Content-Type`。

如果你没有设置`Content-Type`，大多数的客户端会默认将`Contetn-Type`设置为`application/x-www-form-urlencoded`。

举个栗子：如果你要使用jQuery和[.ajax() method][jquery-ajax]来发送`json`编码格式的数据，那你应该确保`Content-Type`被设置为`application/json`。



---

## Setting the parsers

The default set of parsers may be set globally, using the `DEFAULT_PARSER_CLASSES` setting. For example, the following settings would allow only requests with `JSON` content, instead of the default of JSON or form data.

默认的Parsers通过`DEFAULT_PARSER_CLASSES`被设置为全局配置。举个栗子：如果你只想接收`JSON`内容的请求而不再接收`Form`内容的请求，那你可以像下面这样设置。

```Python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    )
}
```

You can also set the parsers used for an individual view, or viewset, using the `APIView` class-based views.

你也可以为view单独设置parser class。

```Python
from rest_framework.parsers import JSONParser
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    """
    A view that can accept POST requests with JSON content.
    """
    parser_classes = (JSONParser,)

    def post(self, request, format=None):
        return Response({'received data': request.data})
```

Or, if you're using the `@api_view` decorator with function based views.

```Python
from rest_framework.decorators import api_view
from rest_framework.decorators import parser_classes
from rest_framework.parsers import JSONParser

@api_view(['POST'])
@parser_classes((JSONParser,))
def example_view(request, format=None):
    """
    A view that can accept POST requests with JSON content.
    """
    return Response({'received data': request.data})
```

---

# API Reference

## JSONParser

Parses `JSON` request content.

**.media_type**: `application/json`



解析`JSON`请求内容。



## FormParser

Parses HTML form content.  `request.data` will be populated with a `QueryDict` of data.

You will typically want to use both `FormParser` and `MultiPartParser` together in order to fully support HTML form data.

**.media_type**: `application/x-www-form-urlencoded`



解析HTML表单内容。`request.data`将会以`QueryDict`的形式展现。

通常情况下，你需要同时使用`FormParser`和`MultiPartParser`来解析HTML表单数据。



## MultiPartParser

Parses multipart HTML form content, which supports file uploads.  Both `request.data` will be populated with a `QueryDict`.

You will typically want to use both `FormParser` and `MultiPartParser` together in order to fully support HTML form data.

**.media_type**: `multipart/form-data`



解析多部分HTML表单内容，支持文件上传。`request.data`将会以`QueryDict`的形式展示。

通常情况下，你需要同时使用`FormParser`和`MultiPartParser`来解析HTML表单数据。



## FileUploadParser

Parses raw file upload content.  The `request.data` property will be a dictionary with a single key `'file'` containing the uploaded file.

If the view used with `FileUploadParser` is called with a `filename` URL keyword argument, then that argument will be used as the filename.

If it is called without a `filename` URL keyword argument, then the client must set the filename in the `Content-Disposition` HTTP header.  For example `Content-Disposition: attachment; filename=upload.jpg`.

**.media_type**: `*/*`



解析原始文件上传内容。`request.data`是一个dict格式数据，该dict的key为`'file'`，value为上传的文件内容。

如果处理文件上传的View中包含`filename`URL关键字，那么该参数将被用来作为上传文件的名字。

如果处理文件上传的View中不包含`filename`URL关键字，那么客户端必须在HTTP header设置`Content-Disposition`。比如像这样：`Content-Disposition: attachment; filename=upload.jpg`。



##### Notes:

* The `FileUploadParser` is for usage with native clients that can upload the file as a raw data request.  For web-based uploads, or for native clients with multipart upload support, you should use the `MultiPartParser` parser instead.

* Since this parser's `media_type` matches any content type, `FileUploadParser` should generally be the only parser set on an API view.

* `FileUploadParser` respects Django's standard `FILE_UPLOAD_HANDLERS` setting, and the `request.upload_handlers` attribute.  See the [Django documentation][upload-handlers] for more details.

  ​

* `FileUploadParser`通常用于本地客户端使用原始数据上传文件的情况。但基于Web上传文件或者本地客户端多部分上传文件的情况，你应该使用`MultiParser`。

* 因为`FileUploadParser`的`media_type`能够匹配所有内容类型，所以如果你设置了某个API view的解析器为`FileUploadParser`后，就不要添加其他的解析器了。（但你没有使用`FileUploadParser`，那你应该根据实际情况选择解析器）

* `FileUploadParser`遵循Django的标准`FILE_UPLOAD_HANDLERS`配置和`request.upload_handlers`属性。

  ​

##### Basic usage example:

```Python
# views.py
class FileUploadView(views.APIView):
    parser_classes = (FileUploadParser,)

    def put(self, request, filename, format=None):
        file_obj = request.data['file']
        # ...
        # do some stuff with uploaded file
        # ...
        return Response(status=204)

# urls.py
urlpatterns = [
    # ...
    url(r'^upload/(?P<filename>[^/]+)$', FileUploadView.as_view())
]
```

---

# Custom parsers

To implement a custom parser, you should override `BaseParser`, set the `.media_type` property, and implement the `.parse(self, stream, media_type, parser_context)` method.

The method should return the data that will be used to populate the `request.data` property.

如果你想要自定义一个解析器，那该解析器要继承`BaseParser`，设置`.media_type`属性，并实现`.parser(self, stream, media_type, parser_context)`方法。

`.parser（）`方法返回的数据应该可以通过`request.data`获取到。

The arguments passed to `.parse()` are:



### stream

A stream-like object representing the body of the request.

请求内容的流对象形式。



### media_type

Optional.  If provided, this is the media type of the incoming request content.

Depending on the request's `Content-Type:` header, this may be more specific than the renderer's `media_type` attribute, and may include media type parameters.  For example `"text/plain; charset=utf-8"`.

该参数为可选，如果有，则代表请求内容的媒体类型。

取决于请求头中的`Content-Type:`，相比于renderer的`media_type`，parser的`media_type`更加特殊，parser的`media_type`可能包括媒体类型的更多信息。比如：`"text/plain；charset=utf-8"`。



### parser_context

Optional.  If supplied, this argument will be a dictionary containing any additional context that may be required to parse the request content.

By default this will include the following keys: `view`, `request`, `args`, `kwargs`.

该参数可选，如果提供该参数，则是一个包含了用于解析请求内容所需要的上下文的字典。

默认包含这些key：`view`、`request`、`args`、`kwargs`。



## Example

The following is an example plaintext parser that will populate the `request.data` property with a string representing the body of the request.

```Python
class PlainTextParser(BaseParser):
    """
    Plain text parser.
    """
    media_type = 'text/plain'

    def parse(self, stream, media_type=None, parser_context=None):
        """
        Simply return a string representing the body of the request.
        """
        return stream.read()
```

---

# Third party packages

The following third party packages are also available.

一些用于解析请求的第三方包。

## YAML

[REST framework YAML][rest-framework-yaml] provides [YAML][yaml] parsing and rendering support. It was previously included directly in the REST framework package, and is now instead supported as a third-party package.

REST framework YAML 提供了对YAML格式的解析与渲染支持。之前是REST framework的一部分，现在作为一个独立的第三方包提供相关支持。

#### Installation & configuration

Install using pip.

    $ pip install djangorestframework-yaml

Modify your REST framework settings.

```Python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_yaml.parsers.YAMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_yaml.renderers.YAMLRenderer',
    ),
}
```



## XML

[REST Framework XML][rest-framework-xml] provides a simple informal XML format. It was previously included directly in the REST framework package, and is now instead supported as a third-party package.

REST framework XML 提供了XML格式的解析和渲染功能。

#### Installation & configuration

Install using pip.

    $ pip install djangorestframework-xml

Modify your REST framework settings.

```Python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_xml.parsers.XMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_xml.renderers.XMLRenderer',
    ),
}
```



## MessagePack

[MessagePack]() is a fast, efficient binary serialization format.  [Juan Riaza]() maintains the [djangorestframework-msgpack]() package which provides MessagePack renderer and parser support for REST framework.

MessagePack 支持二进制格式数据的解析与渲染。该包由Juan Riaza提供维护。



## CamelCase JSON

[djangorestframework-camel-case]() provides camel case JSON renderers and parsers for REST framework.  This allows serializers to use Python-style underscored field names, but be exposed in the API as Javascript-style camel case field names.  It is maintained by [Vitaly Babiy][vbabiy].

django-rest-framework-camel-case提供了驼峰样式的JSON格式数据的解析和渲染功能。这样，你可以在你的代码中使用Python风格的下划线命名方式，而客户端请求API得到的数据却是驼峰命名风格的。该包由Vitaly Babiy维护。

[jquery-ajax]: http://api.jquery.com/jQuery.ajax/
[cite]: https://groups.google.com/d/topic/django-developers/dxI4qVzrBY4/discussion
[upload-handlers]: https://docs.djangoproject.com/en/stable/topics/http/file-uploads/#upload-handlers
[rest-framework-yaml]: http://jpadilla.github.io/django-rest-framework-yaml/
[rest-framework-xml]: http://jpadilla.github.io/django-rest-framework-xml/
[yaml]: http://www.yaml.org/
[messagepack]: https://github.com/juanriaza/django-rest-framework-msgpack
[juanriaza]: https://github.com/juanriaza
[vbabiy]: https://github.com/vbabiy
[djangorestframework-msgpack]: https://github.com/juanriaza/django-rest-framework-msgpack
[djangorestframework-camel-case]: https://github.com/vbabiy/djangorestframework-camel-case

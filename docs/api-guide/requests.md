source: request.py

# Requests

> If you're doing REST-based web service stuff ... you should ignore request.POST.
>
> &mdash; Malcom Tredinnick, [Django developers group][cite]
>
> 如果你想基于 REST 来开发 web 服务，那么你应该忽略 request.POST 。

REST framework's `Request` class extends the standard `HttpRequest`, adding support for REST framework's flexible request parsing and request authentication.

REST framework 的`Request`对 Django 的`HttpRequest`进行了扩展，支持灵活的请求解析与请求认证。

---

# Request parsing

请求解析

REST framework's Request objects provide flexible request parsing that allows you to treat requests with JSON data or other media types in the same way that you would normally deal with form data.

REST framework 的 Request Objects 提供了灵活的请求解析，你可以使用 JSON 或者其他的媒体类型来处理request，就像我们之前处理表单数据一样简单。



## .data

`request.data` returns the parsed content of the request body.  This is similar to the standard `request.POST` and `request.FILES` attributes except that:

* It includes all parsed content, including *file and non-file* inputs.
* It supports parsing the content of HTTP methods other than `POST`, meaning that you can access the content of `PUT` and `PATCH` requests.
* It supports REST framework's flexible request parsing, rather than just supporting form data.  For example you can handle incoming JSON data in the same way that you handle incoming form data.

For more details see the [parsers documentation].

`request.data`返回的结果是经过解析的请求体。它和 Django 中的`request.POST` 和`request.FILES` 属性一样，但是要注意以下几点：

* `request.data`包括所有解析的内容，包括文件和非文件的输入；
* `request.data`不仅可以解析`POST`方法，还可以解析其他HTTP方法，也就是说你可以通过`request.data`访问`PUT`和`PATCH` 的请求内容；
* `request.data`支持请求的灵活解析，而不仅仅是支持处理表单数据。比如说，你可以像处理表单数据一样处理JSON数据。



## .query_params

`request.query_params` is a more correctly named synonym for `request.GET`.

相对于 `request.GET`来说，`request.query_params`可能是更加准确的表述。

For clarity inside your code, we recommend using `request.query_params` instead of the Django's standard `request.GET`. Doing so will help keep your codebase more correct and obvious - any HTTP method type may include query parameters, not just `GET` requests.

除了GET请求，其他的请求方式也会有查询参数（query_params），所以我们建议你使用`request.query_params`，而不是Django所提供的`request.GET`，这样更能突显你的意图，让你的代码更加清晰明了。



## .parsers

The `APIView` class or `@api_view` decorator will ensure that this property is automatically set to a list of `Parser` instances, based on the `parser_classes` set on the view or based on the `DEFAULT_PARSER_CLASSES` setting.

基于view的`parser_classes`或者`DEFAULT_PARSER_CLASSES`，`APIView`classes 和 `@api_view`装饰器将确保将该属性（.parsers）自动设置给`Parser`instances。

You won't typically need to access this property.

但是通常情况下，你几乎不会用到这个属性。

---

**Note:** If a client sends malformed content, then accessing `request.data` may raise a `ParseError`.  By default REST framework's `APIView` class or `@api_view` decorator will catch the error and return a `400 Bad Request` response.

值得注意的是：如果客户端发送格式错误的内容，那你在访问`request.data`的时候将会抛出`ParserError`。而`APIView`classes和`@api_view`装饰器在默认情况下，会捕获该错误，并响应`400 Bad Request`。

If a client sends a request with a content-type that cannot be parsed then a `UnsupportedMediaType` exception will be raised, which by default will be caught and return a `415 Unsupported Media Type` response.

如果客户端发送的内容类型不能够被解析，那么将会抛出`UnsupportMediaType`异常，默认情况下将会响应`415 Unsupported Media Type`。

---

# Content negotiation

协商响应内容

The request exposes some properties that allow you to determine the result of the content negotiation stage. This allows you to implement behaviour such as selecting a different serialisation schemes for different media types.

返回内容协商选择的渲染器、渲染的媒体格式，而这些结果可以由你来决定。这样，你就可以根据实际的需求，渲染适合你需求的响应。比如不同的请求接收的媒体类型不同，你可以为不同的媒体类型设置不同的序列化方案来序列化同一个对象。

## .accepted_renderer

The renderer instance what was selected by the content negotiation stage.

返回内容协商选择的渲染器实例

## .accepted_media_type

A string representing the media type that was accepted by the content negotiation stage.

返回内容协商选择的客户端接受的媒体类型（字符串形式）

---

# Authentication

身份认证

REST framework provides flexible, per-request authentication, that gives you the ability to:

REST framework提供了丰富的身份认证方案：

* Use different authentication policies for different parts of your API.

  可以为不同的API设置不同的身份认证方案


* Support the use of multiple authentication policies.

  支持多种身份认证方案


* Provide both user and token information associated with the incoming request.

* 提供与接收到的请求相关的user和token信息

  ​

## .user

`request.user` typically returns an instance of `django.contrib.auth.models.User`, although the behavior depends on the authentication policy being used.

在`request`通过身份认证后，`request.user`通常会返回`django.contrib.auth.User`的一个实例，但它也取决于你使用的身份认证方案。

If the request is unauthenticated the default value of `request.user` is an instance of `django.contrib.auth.models.AnonymousUser`.

如果`request`没有通过身份认证，那么`request.user`将会返回`django.contrib.auth.models.AnonymousUser`的一个实例。

For more details see the [authentication documentation].

更多细节请参考authentication的文档。



## .auth

`request.auth` returns any additional authentication context.  The exact behavior of `request.auth` depends on the authentication policy being used, but it may typically be an instance of the token that the request was authenticated against.

在`request`通过身份认证的情况下，`request.auth`通常返回的是token的一个实例。但它也取决于你使用的身份认证方案。

If the request is unauthenticated, or if no additional context is present, the default value of `request.auth` is `None`.

如果`request`没有通过身份认证，或者没有附加的身份认证上下文，那么`reques.auth`将返回`None`。

For more details see the [authentication documentation].



## .authenticators

The `APIView` class or `@api_view` decorator will ensure that this property is automatically set to a list of `Authentication` instances, based on the `authentication_classes` set on the view or based on the `DEFAULT_AUTHENTICATORS` setting.

通过在`setting`中配置`DEFAULT_AUTHENTICATORS`或在view中设置`authentication_class`来确保`APIView`和`@api_view`把`.authenticators`属性自动设置到`Authentication`实例上。

You won't typically need to access this property.

不过，你几乎不会用到这个属性。

---

**Note:** You may see a `WrappedAttributeError` raised when calling the `.user` or `.auth` properties. These errors originate from an authenticator as a standard `AttributeError`, however it's necessary that they be re-raised as a different exception type in order to prevent them from being suppressed by the outer property access. Python will not recognize that the `AttributeError` orginates from the authenticator and will instaed assume that the request object does not have a `.user` or `.auth` property. The authenticator will need to be fixed.

值得注意的是：在你调用`.user`或者`.auth`属性时，你可能会遇到`WrappedAttributeError`。那是因为`authenticator`发生了`AttributeError`。

（那为什么不直接抛出`AttributeError`而是抛出`WrappedAttributerError`呢）因为当外部属性访问时，如果发生错误（即访问不存在的属性时），也会抛出`AttributeError`。

Python不会区分这两种不同原因导致的`AttributeError`，只会把他们都当做`request`没有`.user`和`.auth`属性。

所以为了区分这两种`AttributeError`，由于`Authenticator`引发的`AttributeError`将会抛出`WrappedAttributeError`，那么此时你应该修改一下你的`authenticator`。

---

# Browser enhancements

浏览器增强功能

REST framework supports a few browser enhancements such as browser-based `PUT`, `PATCH` and `DELETE` forms.

REST framework 支持一些基于浏览器的增强功能，比如基于浏览器的`PUT`、`PATCH`和`DELETE`表单。



## .method

`request.method` returns the **uppercased** string representation of the request's HTTP method.

`request.method`返回`request`的HTTP method，为大写字符串格式。

Browser-based `PUT`, `PATCH` and `DELETE` forms are transparently supported.

完全支持`PUT`、`PATCH`和`DELETE`。

For more information see the [browser enhancements documentation].

更多详情请查看[browser enhancements documentation]。



## .content_type

`request.content_type`, returns a string object representing the media type of the HTTP request's body, or an empty string if no media type was provided.

`request.content_type`返回`request`请求体中的`media type`（字符串格式），如果请求体中没有`media type`，那么将会返回一个空字符串。

You won't typically need to directly access the request's content type, as you'll normally rely on REST framework's default request parsing behavior.

你基本上不会直接访问`request's content type`，，因为REST framework默认的请求解析功能已经完成了与它相关的一系列操作。

If you do need to access the content type of the request you should use the `.content_type` property in preference to using `request.META.get('HTTP_CONTENT_TYPE')`, as it provides transparent support for browser-based non-form content.

如果你需要访问`request`的`content type`属性，那么你应该使用`request.content_type`而不是`request.META.get('HTTP_CONTENT_TYPE')`，因为`.content_type`完全支持基于浏览器的非表单内容。

For more information see the [browser enhancements documentation].

更多详细信息请参考[browser enhancenments documentation]。



## .stream

`request.stream` returns a stream representing the content of the request body.

`request.stream`返回流形式的request's body 的内容。

You won't typically need to directly access the request's content, as you'll normally rely on REST framework's default request parsing behavior.

通常情况下，你不需要直接调用该属性，因为REST framework默认的请求解析功能已经完成了与它相关的操作。

---

# Standard HttpRequest attributes

As REST framework's `Request` extends Django's `HttpRequest`, all the other standard attributes and methods are also available.  For example the `request.META` and `request.session` dictionaries are available as normal.

REST framework的`Request`扩展了Django 的`HttpRequest`，（也就是说REST framework 中的`request`包含Django的`HttpRequest`）所以其他的标准`HttpRequest`属性仍然可用。

比如``request.META`和`request.session`可以像在Django中一样正常使用。

Note that due to implementation reasons the `Request` class does not inherit from `HttpRequest` class, but instead extends the class using composition.

但是请注意，REST framework中的`request`对Django 的`HttpRequest`的扩展并不是通过继承`HttpRequest`，而是通过组合了更多其他功能而形成REST framework中的`request`，从而对`HttpRequest`进行了扩展。


[cite]: https://groups.google.com/d/topic/django-developers/dxI4qVzrBY4/discussion
[parsers documentation]: parsers.md
[authentication documentation]: authentication.md
[browser enhancements documentation]: ../topics/browser-enhancements.md

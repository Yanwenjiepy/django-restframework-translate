source: request.py

# Requests

> If you're doing REST-based web service stuff ... you should ignore request.POST.
>
> &mdash; Malcom Tredinnick, [Django developers group][cite]
>
> 如果你想基于 REST 来开发 web 服务，那么你应该忽略 request.POST 。

REST framework's `Request` class extends the standard `HttpRequest`, adding support for REST framework's flexible request parsing and request authentication.

REST framework 的`Request`扩展了 Django 的`HttpRequest`，并且支持灵活的请求解析与请求认证。

---

# Request parsing

REST framework's Request objects provide flexible request parsing that allows you to treat requests with JSON data or other media types in the same way that you would normally deal with form data.

REST framework 的 Request Objects 提供了灵活的请求解析，它能够让你使用 JSON 数据格式或者其他媒体类型处理请求的方式和你之前处理表单数据的方式一样简单。

## .data

`request.data` returns the parsed content of the request body.  This is similar to the standard `request.POST` and `request.FILES` attributes except that:

* It includes all parsed content, including *file and non-file* inputs.
* It supports parsing the content of HTTP methods other than `POST`, meaning that you can access the content of `PUT` and `PATCH` requests.
* It supports REST framework's flexible request parsing, rather than just supporting form data.  For example you can handle incoming JSON data in the same way that you handle incoming form data.

For more details see the [parsers documentation].

`request.data`返回的结果是经过解析的请求体。它和 Django 中的`request.POST` 和`request.FILES` 属性一样，但是要注意以下几点：

* `request.data`包括所有解析的内容，包括文件和非文件的输入；
* `request.data`可以解析除`POST`以外的其他HTTP方法的内容，也就是说你可以通过`request.data`访问`PUT`和`PATCH` 的请求内容；
* `request.data`支持请求的灵活解析，而不仅仅是支持处理表单数据。比如说你在处理传入的JSON数据时的方式可以像你处理传入的表单数据的方式一样。

## .query_params

`request.query_params` is a more correctly named synonym for `request.GET`.

相对于 `request.GET`来说，`request.query_params`可能是更加准确的表述。

For clarity inside your code, we recommend using `request.query_params` instead of the Django's standard `request.GET`. Doing so will help keep your codebase more correct and obvious - any HTTP method type may include query parameters, not just `GET` requests.

为了让你的

## .parsers

The `APIView` class or `@api_view` decorator will ensure that this property is automatically set to a list of `Parser` instances, based on the `parser_classes` set on the view or based on the `DEFAULT_PARSER_CLASSES` setting.

You won't typically need to access this property.

---

**Note:** If a client sends malformed content, then accessing `request.data` may raise a `ParseError`.  By default REST framework's `APIView` class or `@api_view` decorator will catch the error and return a `400 Bad Request` response.

If a client sends a request with a content-type that cannot be parsed then a `UnsupportedMediaType` exception will be raised, which by default will be caught and return a `415 Unsupported Media Type` response.

---

# Content negotiation

The request exposes some properties that allow you to determine the result of the content negotiation stage. This allows you to implement behaviour such as selecting a different serialisation schemes for different media types.

## .accepted_renderer

The renderer instance what was selected by the content negotiation stage.

## .accepted_media_type

A string representing the media type that was accepted by the content negotiation stage.

---

# Authentication

REST framework provides flexible, per-request authentication, that gives you the ability to:

* Use different authentication policies for different parts of your API.
* Support the use of multiple authentication policies.
* Provide both user and token information associated with the incoming request.

## .user

`request.user` typically returns an instance of `django.contrib.auth.models.User`, although the behavior depends on the authentication policy being used.

If the request is unauthenticated the default value of `request.user` is an instance of `django.contrib.auth.models.AnonymousUser`.

For more details see the [authentication documentation].

## .auth

`request.auth` returns any additional authentication context.  The exact behavior of `request.auth` depends on the authentication policy being used, but it may typically be an instance of the token that the request was authenticated against.

If the request is unauthenticated, or if no additional context is present, the default value of `request.auth` is `None`.

For more details see the [authentication documentation].

## .authenticators

The `APIView` class or `@api_view` decorator will ensure that this property is automatically set to a list of `Authentication` instances, based on the `authentication_classes` set on the view or based on the `DEFAULT_AUTHENTICATORS` setting.

You won't typically need to access this property.

---

**Note:** You may see a `WrappedAttributeError` raised when calling the `.user` or `.auth` properties. These errors originate from an authenticator as a standard `AttributeError`, however it's necessary that they be re-raised as a different exception type in order to prevent them from being suppressed by the outer property access. Python will not recognize that the `AttributeError` orginates from the authenticator and will instaed assume that the request object does not have a `.user` or `.auth` property. The authenticator will need to be fixed.

---

# Browser enhancements

REST framework supports a few browser enhancements such as browser-based `PUT`, `PATCH` and `DELETE` forms.

## .method

`request.method` returns the **uppercased** string representation of the request's HTTP method.

Browser-based `PUT`, `PATCH` and `DELETE` forms are transparently supported.

For more information see the [browser enhancements documentation].

## .content_type

`request.content_type`, returns a string object representing the media type of the HTTP request's body, or an empty string if no media type was provided.

You won't typically need to directly access the request's content type, as you'll normally rely on REST framework's default request parsing behavior.

If you do need to access the content type of the request you should use the `.content_type` property in preference to using `request.META.get('HTTP_CONTENT_TYPE')`, as it provides transparent support for browser-based non-form content.

For more information see the [browser enhancements documentation].

## .stream

`request.stream` returns a stream representing the content of the request body.

You won't typically need to directly access the request's content, as you'll normally rely on REST framework's default request parsing behavior.

---

# Standard HttpRequest attributes

As REST framework's `Request` extends Django's `HttpRequest`, all the other standard attributes and methods are also available.  For example the `request.META` and `request.session` dictionaries are available as normal.

Note that due to implementation reasons the `Request` class does not inherit from `HttpRequest` class, but instead extends the class using composition.


[cite]: https://groups.google.com/d/topic/django-developers/dxI4qVzrBY4/discussion
[parsers documentation]: parsers.md
[authentication documentation]: authentication.md
[browser enhancements documentation]: ../topics/browser-enhancements.md

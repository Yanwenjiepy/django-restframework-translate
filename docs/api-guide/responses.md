source: response.py

# Responses

> Unlike basic HttpResponse objects, TemplateResponse objects retain the details of the context that was provided by the view to compute the response.  The final output of the response is not computed until it is needed, later in the response process.
>
> 与HttpResponse对象不同，TemplateResponse对象保留了view提供的用于响应的模板与上下文的详细信息，只有在最终要输出响应时，才会将模板与上下文组合，计算出最终的响应结果。（而HttpResponse则是直接将模板与上下文组合计算出最终响应结果，无论此时是否要输出响应）
>
> &mdash; [Django documentation][cite]

REST framework supports HTTP content negotiation by providing a `Response` class which allows you to return content that can be rendered into multiple content types, depending on the client request.

`Response` class可以根据请求来返回多种不同类型的响应内容。（因为内容协商机制发挥了作用）

The `Response` class subclasses Django's `SimpleTemplateResponse`.  `Response` objects are initialised with data, which should consist of native Python primitives.  REST framework then uses standard HTTP content negotiation to determine how it should render the final response content.

`Response`class是Django的`SimpleTemplateResponse`的子类。`Response`对象由Python类型的数据初始化，REST framework将通过标准的HTTP 内容协商来确定如何渲染最终的响应内容。

There's no requirement for you to use the `Response` class, you can also return regular `HttpResponse` or `StreamingHttpResponse` objects from your views if required.  Using the `Response` class simply provides a nicer interface for returning content-negotiated Web API responses, that can be rendered to multiple formats.

当然，如果你不想使用REST framework的`Response`，你也可以使用`HttpResponse`或者`StreamingHttpResponse`，一切都可以根据实际的需求来灵活的更改，REST framework 的`Response`为返回经过内容协商的Web API Response提供了一个很易用的接口。

Unless you want to heavily customize REST framework for some reason, you should always use an `APIView` class or `@api_view` function for views that return `Response` objects.  Doing so ensures that the view can perform content negotiation and select the appropriate renderer for the response, before it is returned from the view.

除非你想根据自己的需求大幅的更改REST framework，否则你的view都应该使用`APIView`或者`@api_view`，这样可以确保通过内容协商选择合适的渲染器来渲染响应并返回。

---

# Creating responses

## Response()

**Signature:** `Response(data, status=None, template_name=None, headers=None, content_type=None)`

Unlike regular `HttpResponse` objects, you do not instantiate `Response` objects with rendered content.  Instead you pass in unrendered data, which may consist of any Python primitives.

`Response`不像`HttpResponse`那样需要传入渲染的内容才能初始化`HttpResponse`对象，你只需要把未经渲染的数据（Python类型）传入即可。

The renderers used by the `Response` class cannot natively handle complex datatypes such as Django model instances, so you need to serialize the data into primitive datatypes before creating the `Response` object.

`Response`所使用的渲染器无法处理复杂的数据类型，比如Django的model 实例，所以你需要先将数据序列化为简单的数据类型（dict最常用）然后在传入`Response`。

You can use REST framework's `Serializer` classes to perform this data serialization, or use your own custom serialization.

你可以使用REST framework的`Serializer`classes或者自定义序列化器来完成数据的序列化操作。

Arguments:

* `data`: The serialized data for the response.
* 已经序列化的数据


* `status`: A status code for the response.  Defaults to 200.  See also [status codes][statuscodes].
* 该响应的状态码，默认为200


* `template_name`: A template name to use if `HTMLRenderer` is selected.
* 使用`HTMLRenderer`渲染器时，所使用的template 名字


* `headers`: A dictionary of HTTP headers to use in the response.
* response中的HTTP headers（dict形式）


* `content_type`: The content type of the response.  Typically, this will be set automatically by the renderer as determined by content negotiation, but there may be some cases where you need to specify the content type explicitly.
* 响应的内容类型。一般由渲染器来根据内容协商的结果设置，但是在某些情况下，可能需要明确的指定内容类型。

---

# Attributes

## .data

The unrendered, serialized data of the response.

已经序列化但是为渲染的数据



## .status_code

The numeric status code of the HTTP response.

响应的数字状态码



## .content

The rendered content of the response.  The `.render()` method must have been called before `.content` can be accessed.

已经渲染过的内容。在你访问`.accessed`方法之前，你需要确保`.render()`方法已经被调用执行。



## .template_name

The `template_name`, if supplied.  Only required if `HTMLRenderer` or some other custom template renderer is the accepted renderer for the response.

只有response所使用的渲染器是`HTMLRenderer`或者其他自定义的模板渲染器（比如你自定义了某个渲染器，来为特殊的页面进行渲染）时，才需要这个参数。



## .accepted_renderer

The renderer instance that will be used to render the response.

Set automatically by the `APIView` or `@api_view` immediately before the response is returned from the view.

用来渲染响应内容的渲染器实例

在view返回response之前，`APIView`和`@api_view`会自动设置该属性



## .accepted_media_type

The media type that was selected by the content negotiation stage.

Set automatically by the `APIView` or `@api_view` immediately before the response is returned from the view.

内容协商确定的媒体类型

在view返回response之前，`APIView`和`@api_view`会自动设置该属性



## .renderer_context

A dictionary of additional context information that will be passed to the renderer's `.render()` method.

Set automatically by the `APIView` or `@api_view` immediately before the response is returned from the view.

传递给渲染器的`.render()`方法的一个字典形式的额外上下文信息

在view返回response之前，`APIView`和`@api_view`会自动设置该属性

---

# Standard HttpResponse attributes

The `Response` class extends `SimpleTemplateResponse`, and all the usual attributes and methods are also available on the response.  For example you can set headers on the response in the standard way:

`Response`继承自`SimpleTemplateResponse`，所以`HttpResponse`的属性和方法都是可用的。比如你可以使用标准的方式来设置响应的headers。

    response = Response()
    response['Cache-Control'] = 'no-cache'



## .render()

**Signature:** `.render()`

As with any other `TemplateResponse`, this method is called to render the serialized data of the response into the final response content.  When `.render()` is called, the response content will be set to the result of calling the `.render(data, accepted_media_type, renderer_context)` method on the `accepted_renderer` instance.

所有的`TemplateResponse`都一样，通过调用此方法来将序列化后的数据渲染为最终的响应内容。由`accepted_renderer`属性返回的渲染器实例调用`.render(data, accepted_method_type, renderer_context)`方法来生成最终的响应内容。

You won't typically need to call `.render()` yourself, as it's handled by Django's standard response cycle.

通常情况下，你不需要自己去调用`.render()`方法，因为Django的standard response cycle会自动处理。

[cite]: https://docs.djangoproject.com/en/stable/stable/template-response/
[statuscodes]: status-codes.md

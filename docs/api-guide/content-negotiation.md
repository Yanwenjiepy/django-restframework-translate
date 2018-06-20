source: negotiation.py

# Content negotiation

> HTTP has provisions for several mechanisms for "content negotiation" - the process of selecting the best representation for a given response when there are multiple representations available.
>
> HTTP为“内容协商”提供了多种机制 -- 可以有多种形式响应时，选择最恰当的那种表现形式。
>
> &mdash; [RFC 2616][cite], Fielding et al.

[cite]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec12.html

Content negotiation is the process of selecting one of multiple possible representations to return to a client, based on client or server preferences.

内容协商就是根据客户端或者服务端的偏好（这里的偏好是指客户端接受的数据格式或服务端响应的数据格式，至于以哪一端为主，实际业务决定）从多种内容响应形式中选择一种最适合的内容形式响应给客户端。



## Determining the accepted renderer

确定渲染器

REST framework uses a simple style of content negotiation to determine which media type should be returned to a client, based on the available renderers, the priorities of each of those renderers, and the client's `Accept:` header.  The style used is partly client-driven, and partly server-driven.

根据可用的渲染器、每个渲染器的优先级以及客户端的请求头中的`Accept`，REST framework将使用简单的内容协商风格来确定到底返回客户端哪种格式的内容。

1. More specific media types are given preference to less specific media types.
2. If multiple media types have the same specificity, then preference is given to based on the ordering of the renderers configured for the given view.

   1、请求头中`Accept`给出多个媒体类型，哪个越具体，则其优先级越高；

   2、如果多个媒体类型一样详细，那么将按照他们的顺序来对他们的优先级进行排序。

For example, given the following `Accept` header:

    application/json; indent=4, application/json, application/yaml, text/html, */*

The priorities for each of the given media types would be:

* `application/json; indent=4`（最详细）
* `application/json`, `application/yaml` and `text/html`（一样详细，按照他们的顺序给优先级排序）
* `*/*`（不具体）

If the requested view was only configured with renderers for `YAML` and `HTML`, then REST framework would select whichever renderer was listed first in the `renderer_classes` list or `DEFAULT_RENDERER_CLASSES` setting.

如果所请求的view只渲染`YAML`和`HTML`，那么REST framework将首先使用`renderer_classes`或者`DEFAULT_RENDERER_CLASSES`中的渲染器。

For more information on the `HTTP Accept` header, see [RFC 2616][accept-header]

更多关于`HTTP Accept`header的信息，请参考[RFC 2616][accept-header]。

---

**Note**: "q" values are not taken into account by REST framework when determining preference.  The use of "q" values negatively impacts caching, and in the author's opinion they are an unnecessary and overcomplicated approach to content negotiation.

值得注意的是：在确定偏好时，REST framework并不会把“q“values考虑进去。因为”q“values会对缓存产生负面影响，考虑”q“values只会让内容协商这件事情更加复杂。

This is a valid approach as the HTTP spec deliberately underspecifies how a server should weight server-based preferences against client-based preferences.

我们认为这样考虑很好，因为HTTP协议并没有考虑服务器端该如何权衡客户端与服务端的偏好。

---

# Custom content negotiation

自定义内容协商

It's unlikely that you'll want to provide a custom content negotiation scheme for REST framework, but you can do so if needed.  To implement a custom content negotiation scheme override `BaseContentNegotiation`.

一般情况下，REST framework提供的内容协商方案已经能够满足你的需求，但如果你想要根据需求自定义内容协商方案的话，你可以重写`BaseContentNegotiation`的方法。

REST framework's content negotiation classes handle selection of both the appropriate parser for the request, and the appropriate renderer for the response, so you should implement both the `.select_parser(request, parsers)` and `.select_renderer(request, renderers, format_suffix)` methods.

你需要在你自定义的内容协商类中实现`.select_parser(request, parsers)`和`.select_renderer(request, renderers, format_suffix)`这两个方法，用来选择适当的解析器来解析请求以及选择适当的渲染器来渲染响应。

The `select_parser()` method should return one of the parser instances from the list of available parsers, or `None` if none of the parsers can handle the incoming request.

如果传入的请求能够被可用的解析器解析，那么`select_parser()`应该返回这个解析器的实例，如果没有解析器能够解析传入的请求，那么应该返回`None`。

The `select_renderer()` method should return a two-tuple of (renderer instance, media type), or raise a `NotAcceptable` exception.

如果能够找到适合的响应渲染器，那么`select_renderer()`方法应该返回一个二元元组，该二元元组包含渲染器实例与渲染的媒体类型，如果无法渲染客户端希望接收的媒体类型，则跑出`NotAcceptable`异常。

## Example

The following is a custom content negotiation class which ignores the client
request when selecting the appropriate parser or renderer.

下面是一个简单的示例，在该示例中，选择解析器与渲染器的时候并没有考虑客户端的请求（实际中，你是需要考虑客户端的请求的）。

```python
from rest_framework.negotiation import BaseContentNegotiation
```

```python
class IgnoreClientContentNegotiation(BaseContentNegotiation):
    def select_parser(self, request, parsers):
        """
        Select the first parser in the `.parser_classes` list.
        """
        return parsers[0]

    def select_renderer(self, request, renderers, format_suffix):
        """
        Select the first renderer in the `.renderer_classes` list.
        """
        return (renderers[0], renderers[0].media_type)
```

## Setting the content negotiation

The default content negotiation class may be set globally, using the `DEFAULT_CONTENT_NEGOTIATION_CLASS` setting.  For example, the following settings would use our example `IgnoreClientContentNegotiation` class.

内容协商类默认是全局设置，可以通过`DEFAULT_CONTENT_NEGOTIATION_CLASS`来将你自定义的内容协商类设置为全局内容协商类。

下面这个例子中，我们把刚才我们自定义的内容协商类设置为全局内容协商类。

```python
REST_FRAMEWORK = {
    'DEFAULT_CONTENT_NEGOTIATION_CLASS': 'myapp.negotiation.IgnoreClientContentNegotiation',
}
```

You can also set the content negotiation used for an individual view, or viewset, using the `APIView` class-based views.

当然你也可以设置只对某个view有效的内容协商类：

```python
from myapp.negotiation import IgnoreClientContentNegotiation
from rest_framework.response import Response
from rest_framework.views import APIView

class NoNegotiationView(APIView):
    """
    An example view that does not perform content negotiation.
    """
    content_negotiation_class = IgnoreClientContentNegotiation

    def get(self, request, format=None):
        return Response({
            'accepted media type': request.accepted_renderer.media_type
        })
```

[accept-header]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html

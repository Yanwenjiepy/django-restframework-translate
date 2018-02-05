# Tutorial 2: Requests and Responses

接下来，我们将接触 DjangoRESTframework 的核心内容。首先介绍几个构成 RESTframework 的基础部分。

## Request objects

REST framewo 中的`Request` object 扩展了 Django 中的 `HttpRequest`， 提供了更为灵活的请求解析。 

`request.data` 属性是`Request` object 的核心功能,，它与 `request.POST`非常相似，但在 Web APIs 的开发中更加有用。

```Python
request.POST  # Only handles form data.  Only works for 'POST' method.
# 只适用于'POST'请求方式，仅处理表单数据。
request.data  # Handles arbitrary data.  Works for 'POST', 'PUT' and 'PATCH' methods.
# 适用于'POST','PUT','PATCH'请求方式，处理任意数据。
```

## Response objects

REST framework also introduces a `Response` object, which is a type of `TemplateResponse` that takes unrendered content and uses content negotiation to determine the correct content type to return to the client.

REST framework 还引入了一个 `Response` 对象，它是`TemplateResponse`类型的一种，它接受未呈现的内容（序列化后的数据）并使用内容协商来确定返回给客户端的正确内容类型（一般是根据客户端的请求来返回相对应的数据）。

```Python
return Response(data)  # Renders to content type as requested by the client.
# 根据客户端的请求渲染返回内容的类型。
```

## Status codes

Using numeric HTTP status codes in your views doesn't always make for obvious reading, and it's easy to not notice if you get an error code wrong.  REST framework provides more explicit identifiers for each status code, such as `HTTP_400_BAD_REQUEST` in the `status` module.  It's a good idea to use these throughout rather than using numeric identifiers.

在views中使用数字HTTP状态码并不总是很容易阅读，并且如果你接收到了一个错误状态码，很容易遗漏。
REST框架为每个状态代码提供更明确的标识符，例如状态模块中的HTTP_400_BAD_REQUEST。
这是一个好主意，而不是使用数字标识符。

## Wrapping API views        包装API views

REST framework provides two wrappers you can use to write API views.

REST框架提供了两个可用于编写API视图的包装器。

1. The `@api_view` decorator for working with function based views.

   `@api_view`装饰器：处理基于views的函数

2. The `APIView` class for working with class-based views.

   APIView 类：处理基于类的views

These wrappers provide a few bits of functionality such as making sure you receive `Request` instances in your view, and adding context to `Response` objects so that content negotiation can be performed.

这些包装提供了一些功能，例如确保在视图中接收`Request`实例，并向`Response`对象添加上下文，以便可以执行内容协商。

The wrappers also provide behaviour such as returning `405 Method Not Allowed` responses when appropriate, and handling any `ParseError` exception that occurs when accessing `request.data` with malformed input.

这些包装器还提供了一些行为，例如在适当的时候返回`405 Method Not Allowed`响应，以及使用格式错误的输入去访问 `request.data` 时发生的任何`ParseError`异常。

## Pulling it all together

Okay, let's go ahead and start using these new components to write a few views.

We don't need our `JSONResponse` class in `views.py` any more, so go ahead and delete that.  Once that's done we can start refactoring our views slightly.

我们不再需要`views.py`中的`JSONResponse`类。 让我们开始逐步重构我们的视图。

```Python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```


```python
@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

Our instance view is an improvement over the previous example.  It's a little more concise, and the code now feels very similar to if we were working with the Forms API.  We're also using named status codes, which makes the response meanings more obvious.

Here is the view for an individual snippet, in the `views.py` module.

```python
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

This should all feel very familiar - it is not a lot different from working with regular Django views.

Notice that we're no longer explicitly tying our requests or responses to a given content type.  `request.data` can handle incoming `json` requests, but it can also handle other formats.  Similarly we're returning response objects with data, but allowing REST framework to render the response into the correct content type for us.

## Adding optional format suffixes to our URLs

To take advantage of the fact that our responses are no longer hardwired to a single content type let's add support for format suffixes to our API endpoints.  Using format suffixes gives us URLs that explicitly refer to a given format, and means our API will be able to handle URLs such as [http://example.com/api/items/4.json][json-url].

Start by adding a `format` keyword argument to both of the views, like so.

    def snippet_list(request, format=None):

and

    def snippet_detail(request, pk, format=None):

Now update the `snippets/urls.py` file slightly, to append a set of `format_suffix_patterns` in addition to the existing URLs.

    from django.conf.urls import url
    from rest_framework.urlpatterns import format_suffix_patterns
    from snippets import views
    
    urlpatterns = [
        url(r'^snippets/$', views.snippet_list),
        url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
    ]
    
    urlpatterns = format_suffix_patterns(urlpatterns)

We don't necessarily need to add these extra url patterns in, but it gives us a simple, clean way of referring to a specific format.

## How's it looking?

Go ahead and test the API from the command line, as we did in [tutorial part 1][tut-1].  Everything is working pretty similarly, although we've got some nicer error handling if we send invalid requests.

We can get a list of all of the snippets, as before.

    http http://127.0.0.1:8000/snippets/

    HTTP/1.1 200 OK
    ...
    [
      {
        "id": 1,
        "title": "",
        "code": "foo = \"bar\"\n",
        "linenos": false,
        "language": "python",
        "style": "friendly"
      },
      {
        "id": 2,
        "title": "",
        "code": "print \"hello, world\"\n",
        "linenos": false,
        "language": "python",
        "style": "friendly"
      }
    ]

We can control the format of the response that we get back, either by using the `Accept` header:

    http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
    http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML

Or by appending a format suffix:

    http http://127.0.0.1:8000/snippets.json  # JSON suffix
    http http://127.0.0.1:8000/snippets.api   # Browsable API suffix

Similarly, we can control the format of the request that we send, using the `Content-Type` header.

    # POST using form data
    http --form POST http://127.0.0.1:8000/snippets/ code="print 123"
    
    {
      "id": 3,
      "title": "",
      "code": "print 123",
      "linenos": false,
      "language": "python",
      "style": "friendly"
    }
    
    # POST using JSON
    http --json POST http://127.0.0.1:8000/snippets/ code="print 456"
    
    {
        "id": 4,
        "title": "",
        "code": "print 456",
        "linenos": false,
        "language": "python",
        "style": "friendly"
    }

If you add a `--debug` switch to the `http` requests above, you will be able to see the request type in request headers.

Now go and open the API in a web browser, by visiting [http://127.0.0.1:8000/snippets/][devserver].

### Browsability

Because the API chooses the content type of the response based on the client request, it will, by default, return an HTML-formatted representation of the resource when that resource is requested by a web browser.  This allows for the API to return a fully web-browsable HTML representation.

Having a web-browsable API is a huge usability win, and makes developing and using your API much easier.  It also dramatically lowers the barrier-to-entry for other developers wanting to inspect and work with your API.

See the [browsable api][browsable-api] topic for more information about the browsable API feature and how to customize it.

## What's next?

In [tutorial part 3][tut-3], we'll start using class-based views, and see how generic views reduce the amount of code we need to write.

[json-url]: http://example.com/api/items/4.json
[devserver]: http://127.0.0.1:8000/snippets/
[browsable-api]: ../topics/browsable-api.md
[tut-1]: 1-serialization.md
[tut-3]: 3-class-based-views.md

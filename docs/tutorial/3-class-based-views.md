# Tutorial 3: Class-based Views

We can also write our API views using class-based views, rather than function based views.  As we'll see this is a powerful pattern that allows us to reuse common functionality, and helps us keep our code [DRY][dry].

我们也可以使用基于类的视图来编写我们的API视图，而不是基于函数的视图。 正如我们将看到的，这是一个强大的模式，使我们可以重用常用功能，并帮助我们保持代码简洁。

## Rewriting our API using class-based views

使用基于类的视图重写我们的API

We'll start by rewriting the root view as a class-based view.  All this involves is a little bit of refactoring of `views.py`.

我们将把之前写好的视图通过基于类的视图来重写，我们将会对`views.py`进行一些重构。

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
```


```python
class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

So far, so good.  It looks pretty similar to the previous case, but we've got better separation between the different HTTP methods.  We'll also need to update the instance view in `views.py`.

到现在为止，我们所做的和之前的例子看起来没有太大的改变，但是我们对不同的HTTP方法进行了更好的分离。 我们还需要更新views.py中的另一个视图。

```python
class SnippetDetail(APIView):
    """
    Retrieve, update or delete a snippet instance.
    """
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

That's looking good.  Again, it's still pretty similar to the function based view right now.

现在看我们刚刚重写完的基于类视图的API，看起来还可以，但是和基于函数的视图API没什么太大变化。

We'll also need to refactor our `snippets/urls.py` slightly now that we're using class-based views.

由于我们重写了视图API，所以我们需要修改一下`snippets/urls.py`里的代码。

```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

Okay, we're done.  If you run the development server everything should be working just as before.

Okey，我们搞定了，像之前那样验证一下是否正常。

## Using mixins

One of the big wins of using class-based views is that it allows us to easily compose reusable bits of behaviour.

使用基于类的视图的一个很大的好处就是它允许我们轻松地构造可重复使用的行为。

The create/retrieve/update/delete operations that we've been using so far are going to be pretty similar for any model-backed API views we create.  Those bits of common behaviour are implemented in REST framework's mixin classes.

我们平时使用的 创建/检索/更新/删除 常见操作其实和我们刚刚创建的API非常相似。 而这些常见的行为早已经在REST框架的mixin类中实现了。

Let's take a look at how we can compose the views by using the mixin classes.  Here's our `views.py` module again.

那么，接下来，让我们看一看如何使用 mixin类 来构建视图。

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

We'll take a moment to examine exactly what's happening here.  We're building our view using `GenericAPIView`, and adding in `ListModelMixin` and `CreateModelMixin`.

我们使用`GenericAPIView`来构建我们的视图，并且添加了 `ListModelMixin`和`CreateModelMixin`。

The base class provides the core functionality, and the mixin classes provide the `.list()` and `.create()` actions.  We're then explicitly binding the `get` and `post` methods to the appropriate actions.  Simple enough stuff so far.

基类提供了我们需要的核心功能，mixin类提供了`.list()` 和`.create()`功能，我们把这俩个功能方别绑定到`get` 和`post` 请求方法上。目前来说，功能都已实现，而且代码就是这么简单。

```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

Pretty similar.  Again we're using the `GenericAPIView` class to provide the core functionality, and adding in mixins to provide the `.retrieve()`, `.update()` and `.destroy()` actions.

像上面一样，我们基于 `GenericAPIView` 类实现核心功能，让后把minxins提供的 `.retrieve()`, `.update()` an和`.destroy()`功能绑定到对应的请求方法上。

## Using generic class-based views

Using the mixin classes we've rewritten the views to use slightly less code than before, but we can go one step further.  REST framework provides a set of already mixed-in generic views that we can use to trim down our `views.py` module even more.

刚刚我们使用mixin 类实现了我们需要的功能并且简化了代码，但我们仍然可以继续简化代码。REST framework 提供了一些已经封装好的generic views，接下来我们利用这些generic views继续简化我们的代码。

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics
```


```python
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```


```python
class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

Wow, that's pretty concise.  We've gotten a huge amount for free, and our code looks like good, clean, idiomatic Django.

现在的代码看起来是不是很简洁。

Next we'll move onto [part 4 of the tutorial][tut-4], where we'll take a look at how we can deal with authentication and permissions for our API.

接下来，让我们看一看如何给我们的API添加身份验证与权限控制。

[dry]: http://en.wikipedia.org/wiki/Don&#39;t_repeat_yourself
[tut-4]: 4-authentication-and-permissions.md

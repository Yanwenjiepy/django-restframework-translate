# Tutorial 6: ViewSets & Routers

视图集和路由器

REST framework includes an abstraction for dealing with `ViewSets`, that allows the developer to concentrate on modeling the state and interactions of the API, and leave the URL construction to be handled automatically, based on common conventions.

REST framework包含`ViewSets`的抽象概念，它允许开发人员专注于对API的状态和交互进行建模，并根据通用约定来自动构造 URL。

`ViewSet` classes are almost the same thing as `View` classes, except that they provide operations such as `read`, or `update`, and not method handlers such as `get` or `put`.

`VIewSet` classes 几乎和`View` classes一样，但是`ViewSets`并不会对请求方法进行处理（比如它并不能处理`.get()`、`.put()`、`.post()`方法），而只是提供诸如：`.list()`、`.update()`、`.create()`等操作。

A `ViewSet` class is only bound to a set of method handlers at the last moment, when it is instantiated into a set of views, typically by using a `Router` class which handles the complexities of defining the URL conf for you.

一个`ViewSet`是一组相关view的逻辑组合，在实例化的时候，我们通常使用`Router` class 来配置URL。

## Refactoring to use ViewSets

使用ViewsSets重构我们的程序

Let's take our current set of views, and refactor them into view sets.

开始使用viewsets重构我们的程序。

First of all let's refactor our `UserList` and `UserDetail` views into a single `UserViewSet`.  We can remove the two views, and replace them with a single class:

我们首先重构我们的`UserList`和`UserDetails` views。我们可以将这俩个view先注释掉，用我们的viewset来代替：

```Python
from rest_framework import viewsets
```

```python
class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    This viewset automatically provides `list` and `detail` actions.
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

Here we've used the `ReadOnlyModelViewSet` class to automatically provide the default 'read-only' operations.  We're still setting the `queryset` and `serializer_class` attributes exactly as we did when we were using regular views, but we no longer need to provide the same information to two separate classes.

我们使用`ReadOnlyModelViewSet`class 来自动提供默认的‘read-only’操作。我们仍然像常规视图一样需要设置`queeryset`和`serializer_class`，但是我们只需要写一次。

Next we're going to replace the `SnippetList`, `SnippetDetail` and `SnippetHighlight` view classes.  We can remove the three views, and again replace them with a single class.

接下来，我们将同样使用viewset来重构我们之前写的`SnippetList`、`SnippetDetail`、`SnippetHighlight`views。

```python
from rest_framework.decorators import detail_route
from rest_framework.response import Response

class SnippetViewSet(viewsets.ModelViewSet):
    """
    This viewset automatically provides `list`, `create`, `retrieve`,
    `update` and `destroy` actions.

    Additionally we also provide an extra `highlight` action.
    """
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                          IsOwnerOrReadOnly,)

    @detail_route(renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

This time we've used the `ModelViewSet` class in order to get the complete set of default read and write operations.

在这里我们使用`ModelViewSet`class，它提供了默认的‘read’和‘write'操作。

Notice that we've also used the `@detail_route` decorator to create a custom action, named `highlight`.  This decorator can be used to add any custom endpoints that don't fit into the standard `create`/`update`/`delete` style.

你应该注意到了，我们还使用`@detail_route`装饰器创建了一个叫做`highlight`的自定义动作。

如果你需要REST framework提供的标准入口风格（`create`、`update`、`delete`）以外的入口，那么可以使用这个装饰器来自定义任意的一个入口。

Custom actions which use the `@detail_route` decorator will respond to `GET` requests by default.  We can use the `methods` argument if we wanted an action that responded to `POST` requests.

我们使用`@detail_route`创建的自定义的功能，默认会响应`GET`方法。如果我们想让该动作响应`POST`方法，我们可以使用方法参数。

The URLs for custom actions by default depend on the method name itself. If you want to change the way url should be constructed, you can include url_path as a decorator keyword argument.

自定义动作的URLs取决于方法（请求方法，如`GET`、`POST`等）名称。如果你想改变URL的构造方式，那么你要把`url_path`作为关键字参数传入装饰器。

## Binding ViewSets to URLs explicitly

将ViewSets明确的与URLs绑定

The handler methods only get bound to the actions when we define the URLConf.

在我们配置URL时，请求方法只会绑定到动作。

To see what's going on under the hood let's first explicitly create a set of views from our ViewSets.

为了搞清楚到底发生了什么，我们先从ViewSets中组建一组views。

In the `snippets/urls.py` file we bind our `ViewSet` classes into a set of concrete views.

在`snippet/urls.py`文件中，就我们将`ViewSet`classes（我们刚刚编辑的`UserViewSet`和`SnippetViewSets`)绑定到一组具体的views。

```python
from snippets.views import SnippetViewSet, UserViewSet, api_root
from rest_framework import renderers

snippet_list = SnippetViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
snippet_detail = SnippetViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy'
})
snippet_highlight = SnippetViewSet.as_view({
    'get': 'highlight'
}, renderer_classes=[renderers.StaticHTMLRenderer])
user_list = UserViewSet.as_view({
    'get': 'list'
})
user_detail = UserViewSet.as_view({
    'get': 'retrieve'
})
```

Notice how we're creating multiple views from each `ViewSet` class, by binding the http methods to the required action for each view.

注意我们是如何从每一个`ViewSet`class 创建多个 views 的，通过把Http方法绑定到每个view所对应的动作上。

Now that we've bound our resources into concrete views, we can register the views with the URL conf as usual.

既然我们已经把资源与对应的view绑定，我们就可以配置我们的URL了。

```python
urlpatterns = format_suffix_patterns([
    url(r'^$', api_root),
    url(r'^snippets/$', snippet_list, name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$', snippet_detail, name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', snippet_highlight, name='snippet-highlight'),
    url(r'^users/$', user_list, name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$', user_detail, name='user-detail')
])
```

## Using Routers

使用路由器

Because we're using `ViewSet` classes rather than `View` classes, we actually don't need to design the URL conf ourselves.  The conventions for wiring up resources into views and urls can be handled automatically, using a `Router` class.  All we need to do is register the appropriate view sets with a router, and let it do the rest.

我们在使用View classes时候，需要自己设计路由，但是使用ViewSet classes时候，我们不在需要自己设计路由。

Router class 可以自动处理 资源与views 、资源与urls之间的关系，我们只需要使用router 注册适当的view sets，其他的工作，Router都会帮我们自己完成。

Here's our re-wired `snippets/urls.py` file.

接下来，我们重写我们的`snippet/urls.py`文件：

```python
from django.conf.urls import url, include
from rest_framework.routers import DefaultRouter
from snippets import views

# Create a router and register our viewsets with it.
router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet)
router.register(r'users', views.UserViewSet)

# The API URLs are now determined automatically by the router.
urlpatterns = [
    url(r'^', include(router.urls))
]
```

Registering the viewsets with the router is similar to providing a urlpattern.  We include two arguments - the URL prefix for the views, and the viewset itself.

使用Router注册viewsets 其实和提供urlpattern其实很相似。我们需要提供俩个参数：views的URL前缀（正则部分的前缀），viewset本身。

The `DefaultRouter` class we're using also automatically creates the API root view for us, so we can now delete the `api_root` method from our `views` module.

我们使用的`DefaultRouter`class也为我们自动创建了API根视图，所以我们现在可以从`view`模块中删除`api_roo`方法。

## Trade-offs between views vs viewsets

到底是选择views还是viewsets

Using viewsets can be a really useful abstraction.  It helps ensure that URL conventions will be consistent across your API, minimizes the amount of code you need to write, and allows you to concentrate on the interactions and representations your API provides rather than the specifics of the URL conf.

viewsets 其实是非常有用的抽象概念。它能够保证URL关系在你的各个API中保持一致，你不必再过分关注URL配置，将你的精力集中在API的交互和表示，并且你不再需要编辑那么多的代码。

That doesn't mean it's always the right approach to take.  There's a similar set of trade-offs to consider as when using class-based views instead of function based views.  Using viewsets is less explicit than building your views individually.

虽然上面使用viewsets的好处很不错，但是并不是在任何时候都要使用viewsets。就像我们选择使用基于类的view还是使用基于函数的view一样需要权衡。相信你也看出来了，使用viewsets没有使用单独的views更加易读，（如果其他的程序员也了解REST framework，那么他能够很好的理解使用viewsets的这段代码的作用，而不了解REST framework的程序员可能就会一头雾水，在实际工作中要考虑团队的协作与其他成员的情况）。

In [part 7][tut-7] of the tutorial we'll look at how we can add an API schema,
and interact with our API using a client library or command line tool.

在[part 7][tut-7] 中，我们将看到如何添加API模式，以及使用客户端库与命令行工具与我们的API进行交互。

[tut-7]: 7-schemas-and-client-libraries.md

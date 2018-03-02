# Tutorial 5: Relationships & Hyperlinked APIs

API之间的关系与超链接API

At the moment relationships within our API are represented by using primary keys.  In this part of the tutorial we'll improve the cohesion and discoverability of our API, by instead using hyperlinking for relationships.

目前我们的API之间的关系通过使用主键来表示。 接下来，我们将改进API的内聚性和可发现性，使用超链接来建立各个API之间的关系。

## Creating an endpoint for the root of our API

为API的根创建一个入口

Right now we have endpoints for 'snippets' and 'users', but we don't have a single entry point to our API.  To create one, we'll use a regular function-based view and the `@api_view` decorator we introduced earlier. In your `snippets/views.py` add:

我们已经在前面为‘snippets'和’users'创建了入口，但是我们没有进入API的入口。

所以我们需要创建一个API入口，我们将使用常规的视图函数与`@api_view`装饰器来创建，在`snippet/views.py`文件中添加如下代码：

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.reverse import reverse
```


```python
@api_view(['GET'])
def api_root(request, format=None):
    return Response({
        'users': reverse('user-list', request=request, format=format),
        'snippets': reverse('snippet-list', request=request, format=format)
    })
```

Two things should be noticed here. First, we're using REST framework's `reverse` function in order to return fully-qualified URLs; second, URL patterns are identified by convenience names that we will declare later on in our `snippets/urls.py`.

这里有俩点值得注意。第一，我们使用REST framework 的`reverse`函数来返回符合要求的URL；

第二，URL模式通过我们在`snippets/urls.py`文件中声明的别名来标识。

## Creating an endpoint for the highlighted snippets

为高亮的代码片段创建一个入口

The other obvious thing that's still missing from our pastebin API is the code highlighting endpoints.

我们的API还存在一个显而易见的问题就是没有创建高亮显示代码的入口。

Unlike all our other API endpoints, we don't want to use JSON, but instead just present an HTML representation.  There are two styles of HTML renderer provided by REST framework, one for dealing with HTML rendered using templates, the other for dealing with pre-rendered HTML.  The second renderer is the one we'd like to use for this endpoint.

与其他API入口不同的是，我们在这里不想将数据以JSON格式呈现出来，我们更希望使用HTML格式呈现。REST framework提供了俩种HTML渲染器，一种是用来处理使用templates呈现HTML格式数据的渲染器，另一种是用来处理预先呈现为HTML格式数据的渲染器。恰巧第二种渲染器符合我们现在的要求。

The other thing we need to consider when creating the code highlight view is that there's no existing concrete generic view that we can use.  We're not returning an object instance, but instead a property of an object instance.

在创建代码片段高亮显示的view时，我们发现，并没有现成的generic view 可供我们使用。

我们将要创建的代码片段高亮显示 view 返回的不应该是一个实例对象，而应该是实例对象的一个属性。

Instead of using a concrete generic view, we'll use the base class for representing instances, and create our own `.get()` method.  In your `snippets/views.py` add:

既然没有generic view 可供我们使用，那我们就通过使用 base view 来创建代码高亮显示 view，并且创建我们自己的`.get`方法。在`snippets/views.py`中添加如下代码：

```python
from rest_framework import renderers
from rest_framework.response import Response

class SnippetHighlight(generics.GenericAPIView):
    queryset = Snippet.objects.all()
    renderer_classes = (renderers.StaticHTMLRenderer,)

    def get(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)
```

As usual we need to add the new views that we've created in to our URLconf.

We'll add a url pattern for our new API root in `snippets/urls.py`:

接下来，到`snippets/urls.py`文件中配置URL：

```python
url(r'^$', views.api_root),
```

And then add a url pattern for the snippet highlights:

```python
url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', views.SnippetHighlight.as_view()),
```
## Hyperlinking our API

超链接我们的API

Dealing with relationships between entities is one of the more challenging aspects of Web API design.  There are a number of different ways that we might choose to represent a relationship:

处理各API之间的关系是Web API设计中更具挑战性的方面之一。下面有很多不同的方式可供我们选择，用来表示各个API之间的关系：

* Using primary keys.
* 使用主键
* Using hyperlinking between entities.
* 在各个API之间使用超链接
* Using a unique identifying slug field on the related entity.
* 在相关的API中使用唯一的标识字段
* Using the default string representation of the related entity.
* 使用相关API的默认字符串表示形式
* Nesting the related entity inside the parent representation.
* 将相关的API嵌套在其父类表示
* Some other custom representation.
* 其他的自定义表示方式

REST framework supports all of these styles, and can apply them across forward or reverse relationships, or apply them across custom managers such as generic foreign keys.

REST framework 支持所有的这些方式，可以通过正向或者反向的关系来使用他们，或者通过自定义managers（比如通用外键）来使用他们。

In this case we'd like to use a hyperlinked style between entities.  In order to do so, we'll modify our serializers to extend `HyperlinkedModelSerializer` instead of the existing `ModelSerializer`.

在这里我们将在各个API之间使用超链接，为此，我们要修改我们之前写的序列化器，它将不在继承`ModelSerializer`而将继承`HyperlinkedModelSerializer`。

The `HyperlinkedModelSerializer` has the following differences from `ModelSerializer`:

`HyperlinkedModelSerializer`与`ModelSerializer`有下面几个不同点：

* It does not include the `id` field by default.

* `HyperlinkedModelSerializer`默认不包含`id`字段。

* It includes a `url` field, using `HyperlinkedIdentityField`.

* 包含`HyperlinkedIdentityField`这个`url`字段。

* Relationships use `HyperlinkedRelatedField`,

  instead of `PrimaryKeyRelatedField`.

* 关系字段使用`HyperlinkedRelatedField`而不再是`PrimaryKeyRelatedField`。

We can easily re-write our existing serializers to use hyperlinking. In your `snippets/serializers.py` add:

将我们的序列化器修改为使用超链接的方式很容易。在`snippets/serializer.py`文件中修改代码如下：

```python
class SnippetSerializer(serializers.HyperlinkedModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')
    highlight = serializers.HyperlinkedIdentityField(view_name='snippet-highlight', format='html')

    class Meta:
        model = Snippet
        fields = ('url', 'id', 'highlight', 'owner',
                  'title', 'code', 'linenos', 'language', 'style')
```


```python
class UserSerializer(serializers.HyperlinkedModelSerializer):
    snippets = serializers.HyperlinkedRelatedField(many=True, view_name='snippet-detail', read_only=True)

    class Meta:
        model = User
        fields = ('url', 'id', 'username', 'snippets')
```

Notice that we've also added a new `'highlight'` field.  This field is of the same type as the `url` field, except that it points to the `'snippet-highlight'` url pattern, instead of the `'snippet-detail'` url pattern.

你会发现，我们添加了一个`highlight`的新字段。该字段与url字段类型相同，不同之处在于它指向的是`snippet-highlight`url模式，而url字段指向的是`snippet-detail`url模式。

Because we've included format suffixed URLs such as `'.json'`, we also need to indicate on the `highlight` field that any format suffixed hyperlinks it returns should use the `'.html'` suffix.

我们在前面已经设置支持使用格式后缀的URLs，所以我们要在`highlight`字段中声明该字段返回的任何超链接后缀都应该使用`.html`后缀。

## Making sure our URL patterns are named

命名URL patterns

If we're going to have a hyperlinked API, we need to make sure we name our URL patterns.  Let's take a look at which URL patterns we need to name.

如果你想使用超链接的API，那么我们需要确保我们命名了我们的URL patterns。让我们看一看那些URL patterns需要命名。

* The root of our API refers to `'user-list'` and `'snippet-list'`.
* 我们的API的根指向`user-list`和`snippet-list`。
* Our snippet serializer includes a field that refers to `'snippet-highlight'`.
* 我们的 snippet serializer 包含一个指向`snippet-highlight`的字段。
* Our user serializer includes a field that refers to `'snippet-detail'`.
* 我们的 user serializer 包含一个指向`snippet-detail`的字段。
* Our snippet and user serializers include `'url'` fields that by default will refer to `'{model_name}-detail'`, which in this case will be `'snippet-detail'` and `'user-detail'`.
* 我们的 snippet serializer 和 user serializer 包含的`url`字段默认指向`{model_name} - detail`，在这里分别指的就是`snippet-detail`和`user-detail`。

After adding all those names into our URLconf, our final `snippets/urls.py` file should look like this:

在URLconf中配置好我们的命名后，我们的`snippets/urls.py`文件应该是这样子的：

```python
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

# API endpoints
urlpatterns = format_suffix_patterns([
    url(r'^$', views.api_root),
    url(r'^snippets/$',
        views.SnippetList.as_view(),
        name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$',
        views.SnippetDetail.as_view(),
        name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$',
        views.SnippetHighlight.as_view(),
        name='snippet-highlight'),
    url(r'^users/$',
        views.UserList.as_view(),
        name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$',
        views.UserDetail.as_view(),
        name='user-detail')
])
```

## Adding pagination

添加分页功能

The list views for users and code snippets could end up returning quite a lot of instances, so really we'd like to make sure we paginate the results, and allow the API client to step through each of the individual pages.

最终返回的代码片段与作者信息可能很多，几乎没有什么人乐意在一页上面看完这么多的数据，所以分页是必须的，并且我们应该允许API client 能够单独浏览每个页面。

We can change the default list style to use pagination, by modifying our `tutorial/settings.py` file slightly. Add the following setting:

我们可以将默认的展示方式修改为使用分页展示的方式，在`tutorial.settings.py`文件中，添加如下代码：

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

Note that settings in REST framework are all namespaced into a single dictionary setting, named `REST_FRAMEWORK`, which helps keep them well separated from your other project settings.

有关REST framework的设置全部在一个名字是`REST_FRAMEWORK`的字典中配置，这有助于与其他的配置区分。

We could also customize the pagination style if we needed too, but in this case we'll just stick with the default.

如果有需要，也可以自定义分页样式，在这里，我们就使用默认的分页样式。

## Browsing the API

在浏览器中测试一下我们的API

If we open a browser and navigate to the browsable API, you'll find that you can now work your way around the API simply by following links.

如果我们在浏览器中访问我们的API，您会发现您可以通过简单的链接访问API。

You'll also be able to see the 'highlight' links on the snippet instances, that will take you to the highlighted code HTML representations.

您还可以看到片段实例上的“高亮”链接，这会将您带到突出显示的代码。

In [part 6][tut-6] of the tutorial we'll look at how we can use ViewSets and Routers to reduce the amount of code we need to build our API.

在[part 6][tut-6] ，我们将会看到使用 ViewSets和 Routers 来减少构建我们的API的代码量。

[tut-6]: 6-viewsets-and-routers.md

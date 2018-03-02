# Tutorial 4: Authentication & Permissions

###身份认证和权限控制

Currently our API doesn't have any restrictions on who can edit or delete code snippets.  We'd like to have some more advanced behavior in order to make sure that:

目前，任何人都可以对我们数据库中的代码片段进行编辑与删除。为了保护我们自己一步一步写下来的代码，我们必须采取一些行动了：

* Code snippets are always associated with a creator.
* 代码的作者应该和属于他的代码之间存在必然的关联。
* Only authenticated users may create snippets.
* 只有通过身份验证的作者，才能创建代码。
* Only the creator of a snippet may update or delete it.
* 只有作者才有更新与删除代码的权利。
* Unauthenticated requests should have full read-only access.
* 未通过身份验证普通游客，只能浏览代码。

## Adding information to our model

We're going to make a couple of changes to our `Snippet` model class.

我们将对`Snippet`模型类进行一些修改。

First, let's add a couple of fields.  One of those fields will be used to represent the user who created the code snippet.  The other field will be used to store the highlighted HTML representation of the code.

我们添加一些字段，有一个字段用来表示用户，其他的字段用来存储突出显示的HTML格式代码。

Add the following two fields to the `Snippet` model in `models.py`.

将下面两个字段添加到 Snippet  model 中。

```python
owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
highlighted = models.TextField()
```

We'd also need to make sure that when the model is saved, that we populate the highlighted field, using the `pygments` code highlighting library.

我们还需要确保在保存模型时，使用pygments代码高亮库来填充我们需要突出显示的字段。

We'll need some extra imports:

导入pygments库：

```python
from pygments.lexers import get_lexer_by_name
from pygments.formatters.html import HtmlFormatter
from pygments import highlight
```

And now we can add a `.save()` method to our model class:

接下来，我们需要给我们的 model class 添加 `.save()`方法：

```python
def save(self, *args, **kwargs):
    """
    Use the `pygments` library to create a highlighted HTML
    representation of the code snippet.
    """
    lexer = get_lexer_by_name(self.language)  # lexer 是词法分析器
    linenos = self.linenos and 'table' or False
    options = self.title and {'title': self.title} or {}
    formatter = HtmlFormatter(style=self.style, linenos=linenos,
                              full=True, **options)
    self.highlighted = highlight(self.code, lexer, formatter)
    super(Snippet, self).save(*args, **kwargs)
```

When that's all done we'll need to update our database tables.

增加上面的代码后，我们需要更新我们的数据库。

Normally we'd create a database migration in order to do that, but for the purposes of this tutorial, let's just delete the database and start again.

正常情况下，应该重新生成迁移文件，然后进行迁移。但是在本教程中，我们就把之前的数据库删除，然后重新开始。

    rm -f db.sqlite3
    rm -r snippets/migrations
    python manage.py makemigrations snippets
    python manage.py migrate

You might also want to create a few different users, to use for testing the API.  The quickest way to do this will be with the `createsuperuser` command.

如果你想创建一些不同的用户，来对我们的API进行测试。那你可以通过 `createsuperuser` 创建一个管理员用户。

    python manage.py createsuperuser

## Adding endpoints for our User models

Now that we've got some users to work with, we'd better add representations of those users to our API.  Creating a new serializer is easy. In `serializers.py` add:

现在我们可以创建几个用户，我们需要把这些用户添加到我们的API中。在`serializers.py`中创建一个新的序列化器。

```python
from django.contrib.auth.models import User
```

```python
class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
```

Because `'snippets'` is a *reverse* relationship on the User model, it will not be included by default when using the `ModelSerializer` class, so we needed to add an explicit field for it.

由于'snippets'与User model是反向关系，所以在继承ModelSerializer类时‘snippets'不会被默认包含，所以我们需要为'snippets'添加一个显式字段(我们在Snippet model 中定义的owner字段，’auth.User‘是‘Snippet’的关联对象，而‘auth.User’可以通过‘snippets’反查到‘Snippet’对象。如果不添加‘snippets’字段，那么‘Snippet’的实例可以访问'User'，但是‘User’的实例却不能访问‘Snippet’)。

We'll also add a couple of views to `views.py`.  We'd like to just use read-only views for the user representations, so we'll use the `ListAPIView` and `RetrieveAPIView` generic class-based views.

对于用户的信息，我们只能查看，所以我们使用`ListAPIView`和`RetrieveAPIView`。

```python
from django.contrib.auth.models import User
```


```python
class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```


```python
class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

Make sure to also import the `UserSerializer` class

记住要导入`UserSerializer` class。

```python
from snippets.serializers import UserSerializer
```

Finally we need to add those views into the API, by referencing them from the URL conf. Add the following to the patterns in `urls.py`.

然后我们需要到`url.py`文件中配置一下我们的url。

```python
url(r'^users/$', views.UserList.as_view()),
url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
```

## Associating Snippets with Users

关联代码片段与用户

Right now, if we created a code snippet, there'd be no way of associating the user that created the snippet, with the snippet instance.  The user isn't sent as part of the serialized representation, but is instead a property of the incoming request.

现在，即使我们使用某个用户的身份来新创建一个代码片段，也无法将代码片段与用户关联。

因为用户不是作为序列化表示的一部分发送的（user实例并没有被序列化），而是作为传入请求的属性（`request.user`）。

The way we deal with that is by overriding a `.perform_create()` method on our snippet views, that allows us to modify how the instance save is managed, and handle any information that is implicit in the incoming request or requested URL.

我们解决这个问题的方法是：在代码片段视图中重写`.perform_create()` 方法，这允许我们修改实例保存的管理方式，并处理传入请求或请求URL中隐含的任何信息。

On the `SnippetList` view class, add the following method:

在`SnippetList` view class 中添加下面的方法：

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

The `create()` method of our serializer will now be passed an additional `'owner'` field, along with the validated data from the request.

现在我们的序列化器的`create()`方法将被传递一个额外的'owner'字段，以及来自请求的验证数据。

## Updating our serializer

Now that snippets are associated with the user that created them, let's update our `SnippetSerializer` to reflect that.  Add the following field to the serializer definition in `serializers.py`:

现在，代码片段与创建它们的用户相关联，接下来更新我们的`SnippetSerializer`。 将以下字段添加到`serializers.py`中：

```python
owner = serializers.ReadOnlyField(source='owner.username')
```

**Note**: Make sure you also add `'owner',` to the list of fields in the inner `Meta` class.

注意：前面更新完`Snippet model `中的 owner 字段后，你是否将 owner 添加到了 `SnippetSerializer` 的 `Meta`class 中。

This field is doing something quite interesting.  The `source` argument controls which attribute is used to populate a field, and can point at any attribute on the serialized instance.  It can also take the dotted notation shown above, in which case it will traverse the given attributes, in a similar way as it is used with Django's template language.

注意我们刚刚所写的这个字段。 `source`参数用来控制哪个属性用于填充字段，并且可以指向序列化实例的任何属性。

也可以采用虚线符号，在这种情况下，它将以与Django的模板语言一样的方式遍历给定的属性。

The field we've added is the untyped `ReadOnlyField` class, in contrast to the other typed fields, such as `CharField`, `BooleanField` etc...  The untyped `ReadOnlyField` is always read-only, and will be used for serialized representations, but will not be used for updating model instances when they are deserialized. We could have also used `CharField(read_only=True)` here.

与`CharField`和`BooleanField`等类型的字段相比，我们所使用的`ReadOnlyField`字段是无类型字段。无类型的`ReadOnlyField`总是只读的，并且将用于序列化表示，但不会用于在反序列化时更新模型实例。我们也可以在这里使用`CharField(read_only=True)`。

## Adding required permissions to views

给views 添加必要的权限

Now that code snippets are associated with users, we want to make sure that only authenticated users are able to create, update and delete code snippets.

现在代码片段与它的作者已经关联起来了，我们想要确保只有作者才有权限 create、update、delete代码片段，那么如何实现呢。

REST framework includes a number of permission classes that we can use to restrict who can access a given view.  In this case the one we're looking for is `IsAuthenticatedOrReadOnly`, which will ensure that authenticated requests get read-write access, and unauthenticated requests get read-only access.

REST framework 包含许多已经定义好的 permission classes，我们可以使用它们来帮助我们限制到底谁可以访问我们的view。在这种情况下，`IsAuthenticatedOrReadOnly`正好满足我们的要求，它将确保经过身份验证的请求获得读写访问权限，未经身份验证的请求将获得只读访问权限。

First add the following import in the views module

首先，在views模块中

导入与permission相关的模块

```python
from rest_framework import permissions
```

Then, add the following property to **both** the `SnippetList` and `SnippetDetail` view classes.

将下面的属性添加到`SnippetList`与`SnippetDetails`view class中。

```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

## Adding login to the Browsable API

登录到我们的API

If you open a browser and navigate to the browsable API at the moment, you'll find that you're no longer able to create new code snippets.  In order to do so we'd need to be able to login as a user.

如果您现在打开浏览器输入之前的网址，你会发现不能够创建新的代码片段了，那是因为我们刚刚设置的权限控制。 为了能够发布新的代码，我们需要能够以用户身份登录。

We can add a login view for use with the browsable API, by editing the URLconf in our project-level `urls.py` file.

我们可以添加一个login view 来登录到我们的API，如何实现呢？自己写一个么？不需要的，我们只需要项目级别的`url.py`中配置URL即可。

Add the following import at the top of the file:

在文件顶部导入相关模块

```python
from django.conf.urls import include
```

And, at the end of the file, add a pattern to include the login and logout views for the browsable API. 

在文件末尾添加登录与登出views 的URL配置。                      

```python
urlpatterns += [
    url(r'^api-auth/', include('rest_framework.urls'),
]
```

The `r'^api-auth/'` part of pattern can actually be whatever URL you want to use.

模式的`r'^ api-auth /'`部分实际上可以是您想要使用的任何URL。

Now if you open up the browser again and refresh the page you'll see a 'Login' link in the top right of the page.  If you log in as one of the users you created earlier, you'll be able to create code snippets again.

现在刷新你的浏览器你就可以在页面顶端看到‘Login'链接。如果你以之前创建的某个用户身份登录，那么你就可以创建代码片段了。

Once you've created a few code snippets, navigate to the '/users/' endpoint, and notice that the representation includes a list of the snippet ids that are associated with each user, in each user's 'snippets' field.

一旦你创建了一些代码片段，请跳转到‘/users/'节点，留意每个用户的’snippets‘字段中，与用户关联的代码片段id的展现形式。

## Object level permissions

对象等级的权限设置

Really we'd like all code snippets to be visible to anyone, but also make sure that only the user that created a code snippet is able to update or delete it.

我们所有的代码片段能够被所有人浏览，但只有作者才能对其代码片段进行编辑。

To do that we're going to need to create a custom permission.

那么我们需要一个自定义权限。

In the snippets app, create a new file, `permissions.py`

在snippets app 中，创建一个名为`permission.py`的新文件。

```python
from rest_framework import permissions
```


```python
class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Write permissions are only allowed to the owner of the snippet.
        return obj.owner == request.user
```

Now we can add that custom permission to our snippet instance endpoint, by editing the `permission_classes` property on the `SnippetDetail` view class:

现在我们可以通过编辑`SnippetDetail`view class中的`permission_classes`属性来将该自定义权限添加到我们的代码片段实例端：

```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                      IsOwnerOrReadOnly,)
```

Make sure to also import the `IsOwnerOrReadOnly` class.

记得导入`IsOwnerOrReadOnly`class。

```python
from snippets.permissions import IsOwnerOrReadOnly
```

Now, if you open a browser again, you find that the 'DELETE' and 'PUT' actions only appear on a snippet instance endpoint if you're logged in as the same user that created the code snippet.

现在，如果你再次打开浏览器，并且以创建代码片段的作者身份登录，你会发现仅有'DELETE'和'PUT'操作在代码片段实例端。

## Authenticating with the API

使用API进行身份验证

Because we now have a set of permissions on the API, we need to authenticate our requests to it if we want to edit any snippets.  We haven't set up any [authentication classes][authentication], so the defaults are currently applied, which are `SessionAuthentication` and `BasicAuthentication`.

因为我们刚刚对我们的API设置了权限，所以如果我们想编辑任何代码片段，就得对请求进行身份验证。

因为我们并没有自定义`authentication class`，所以我们使用的是默认的`SessionAuthentication`和`BasicAuthentication`。

When we interact with the API through the web browser, we can login, and the browser session will then provide the required authentication for the requests.

当我们通过浏览器与API交互时，我们可以登录，然后浏览器会话将为请求提供所需的身份验证。

If we're interacting with the API programmatically we need to explicitly provide the authentication credentials on each request.

如果我们正在以编程方式与API进行交互，那么我们需要在每个请求上明确提供身份验证凭据。

If we try to create a snippet without authenticating, we'll get an error:

如果我们没有进行身份验证就尝试创建代码片段，就发得到以下错误:

    http POST http://127.0.0.1:8000/snippets/ code="print 123"

```json
{
    "detail": "Authentication credentials were not provided."
}
```

We can make a successful request by including the username and password of one of the users we created earlier.

    http -a admin:password123 POST http://127.0.0.1:8000/snippets/ code="print 789"

```json
{
    "id": 1,
    "owner": "admin",
    "title": "foo",
    "code": "print 789",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```

## Summary

We've now got a fairly fine-grained set of permissions on our Web API, and end points for users of the system and for the code snippets that they have created.

现在我们已经在我们的Web API上获得了相当细致的权限集合，以及系统用户端和他们创建的代码片段。

In [part 5][tut-5] of the tutorial we'll look at how we can tie everything together by creating an HTML endpoint for our highlighted snippets, and improve the cohesion of our API by using hyperlinking for the relationships within the system.

在[part 5][tut-5] 中，我们将了解如何通过为突出显示的片段创建HTML端点来将所有内容绑定在一起，并通过对系统内的关系使用超链接来提高API的凝聚力（高内聚）。

[authentication]: ../api-guide/authentication.md
[tut-5]: 5-relationships-and-hyperlinked-apis.md

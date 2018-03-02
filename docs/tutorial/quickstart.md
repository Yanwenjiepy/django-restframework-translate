# Quickstart

快速教程

(如果你想看一看REST framework完成的API到底是什么样子的，那么你可以先看这一部分，而如果你在完成Quickstart后想了解更多细节或者你想直接从头一步一步来完成API，那么你应该从详细的tutourial开始。)

We're going to create a simple API to allow admin users to view and edit the users and groups in the system.

## Project setup

创建项目

Create a new Django project named `tutorial`, then start a new app called `quickstart`.

首先创建一个叫做`tutorial`的项目，在项目中创建一个叫做`quickstart`的应用。

    # Create the project directory
    mkdir tutorial
    cd tutorial
    
    # Create a virtualenv to isolate our package dependencies locally
    virtualenv env
    source env/bin/activate  # On Windows use `env\Scripts\activate`
    
    # Install Django and Django REST framework into the virtualenv
    pip install django
    pip install djangorestframework
    
    # Set up a new project with a single application
    django-admin.py startproject tutorial .  # Note the trailing '.' character
    cd tutorial
    django-admin.py startapp quickstart
    cd ..

The project layout should look like:

项目结构应该是这样的：

    $ pwd
    <some path>/tutorial
    $ find .
    .
    ./manage.py
    ./tutorial
    ./tutorial/__init__.py
    ./tutorial/quickstart
    ./tutorial/quickstart/__init__.py
    ./tutorial/quickstart/admin.py
    ./tutorial/quickstart/apps.py
    ./tutorial/quickstart/migrations
    ./tutorial/quickstart/migrations/__init__.py
    ./tutorial/quickstart/models.py
    ./tutorial/quickstart/tests.py
    ./tutorial/quickstart/views.py
    ./tutorial/settings.py
    ./tutorial/urls.py
    ./tutorial/wsgi.py

It may look unusual that the application has been created within the project directory. Using the project's namespace avoids name clashes with external module (topic goes outside the scope of the quickstart).

我们在项目的目录下创建应用看起来似乎和我们通常进行的操作不太一样。使用项目的名称空间避免了与外部模块的名称冲突（主题不在快速入门的范围内）。

Now sync your database for the first time:

现在同步你的数据库：

    python manage.py migrate
We'll also create an initial user named `admin` with a password of `password123`. We'll authenticate as that user later in our example.

我们先创建一个名字叫做`admin`的用户，并且把他的密码设置为`password123`。我们将在稍后的例子中使用他做身份认证。

    python manage.py createsuperuser --email admin@example.com --username admin
Once you've set up a database and initial user created and ready to go, open up the app's directory and we'll get coding...

一旦你同步完你的数据库并且创建了用户，那么接下来我们就要开始coding了：

## Serializers

序列化器

First up we're going to define some serializers. Let's create a new module named `tutorial/quickstart/serializers.py` that we'll use for our data representations.

首先，我们需要定义一些序列化器。我们在`tutorial/quickstart/`目录下新建一个文件`serializer.py`，这个文件用来对我们的数据进行序列化与反序列化。

```Python
from django.contrib.auth.models import User, Group
from rest_framework import serializers
```


```Python
class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'username', 'email', 'groups')
```


```Python
class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ('url', 'name')
```

Notice that we're using hyperlinked relations in this case, with `HyperlinkedModelSerializer`.  You can also use primary key and various other relationships, but hyperlinking is good RESTful design.

注意，在这里我们使用的是超链接关系，我们定义的序列化器继承于`HyperlinkedModelSerializer`。

当然你也可以使用主键关系或者其他关系，但是对于一个RESTful 风格的API来说，使用hyperlink是很好的选择。

## Views

视图

Right, we'd better write some views then.  Open `tutorial/quickstart/views.py` and get typing.

我们当然还需要编辑一些views。在`tutorial/quickstart/views.py`中添加如下代码：

```Python 
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer
```


```Python
class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
```


```Python
class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```

Rather than write multiple views we're grouping together all the common behavior into classes called `ViewSets`.

我们不再编写多个views，而是将常见的行为进行组合，写入`ViewSets`中。

We can easily break these down into individual views if we need to, but using viewsets keeps the view logic nicely organized as well as being very concise.

如果有需要，我们也可以轻松的将ViewSets分解为多个views，但是使用ViewSets能够保持结构清晰，代码简洁。

## URLs

URL配置（使用routers）

Okay, now let's wire up the API URLs.  On to `tutorial/urls.py`...

好的，现在我们在`tutorial/urls.py`文件中配置我们的URL：

```Python
from django.conf.urls import url, include
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

Because we're using viewsets instead of views, we can automatically generate the URL conf for our API, by simply registering the viewsets with a router class.

因为我们使用的是ViewSets，所以我们可以在router class 中注册我们的viewsets 来自动对我们的API进行URL配置。

Again, if we need more control over the API URLs we can simply drop down to using regular class-based views, and writing the URL conf explicitly.

如果你想对API的URL的进行更详细的配置，那你可以使用基于类的views，然后对URL进行详细的配置。

Finally, we're including default login and logout views for use with the browsable API.  That's optional, but useful if your API requires authentication and you want to use the browsable API.

最后，我们包含了默认的可浏览的 登录与登出views。是否需要这个功能，由你来决定，但是如果你的API需要进行身份验证，并且你想在浏览器上进行测试，那这是极好的。

## Settings

设置

Add `'rest_framework'` to `INSTALLED_APPS`. The settings module will be in `tutorial/settings.py`

在`tutorial/setting.py`中的INSTALLED_APPS中添加'rest_framework'。

```Python 
INSTALLED_APPS = (
    ...
    'rest_framework',
)
```

Okay, we're done.

好了，完成了。

---

## Testing our API

测试我们的API

We're now ready to test the API we've built.  Let's fire up the server from the command line.

开始测试，首先，启动我们的服务。

    python manage.py runserver
We can now access our API, both from the command-line, using tools like `curl`...

现在可以和我们的API进行交互了，比如你可以使用`curl`工具。

    bash: curl -H 'Accept: application/json; indent=4' -u admin:password123 http://127.0.0.1:8000/users/
    {
        "count": 2,
        "next": null,
        "previous": null,
        "results": [
            {
                "email": "admin@example.com",
                "groups": [],
                "url": "http://127.0.0.1:8000/users/1/",
                "username": "admin"
            },
            {
                "email": "tom@example.com",
                "groups": [                ],
                "url": "http://127.0.0.1:8000/users/2/",
                "username": "tom"
            }
        ]
    }

Or using the [httpie][httpie], command line tool...

或者使用httpie。

    bash: http -a admin:password123 http://127.0.0.1:8000/users/

    HTTP/1.1 200 OK
    ...
    {
        "count": 2,
        "next": null,
        "previous": null,
        "results": [
            {
                "email": "admin@example.com",
                "groups": [],
                "url": "http://localhost:8000/users/1/",
                "username": "paul"
            },
            {
                "email": "tom@example.com",
                "groups": [                ],
                "url": "http://127.0.0.1:8000/users/2/",
                "username": "tom"
            }
        ]
    }

Or directly through the browser, by going to the URL `http://127.0.0.1:8000/users/`...

或者打开浏览器，输入`http://127.0.0.1:8000/users/`。

![ Quick start image][image]

If you're working through the browser, make sure to login using the control in the top right corner.

如果使用浏览器，请使用右上角的`admin`进行登录。

Great, that was easy!

好了，就是这么简单！

If you want to get a more in depth understanding of how REST framework fits together head on over to [the tutorial][tutorial], or start browsing the [API guide][guide].

如果您想更深入地了解REST framework如何适用于本教程，或者开始浏览API指南，请点击下面的链接。

[readme-example-api]: ../#example
[image]: ../img/quickstart.png
[tutorial]: 1-serialization.md
[guide]: ../#api-guide
[httpie]: https://github.com/jakubroztocil/httpie#installation

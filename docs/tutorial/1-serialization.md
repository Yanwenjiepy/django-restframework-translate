# Tutorial 1: Serialization

## Introduction

This tutorial will cover creating a simple pastebin code highlighting Web API.  Along the way it will introduce the various components that make up REST framework, and give you a comprehensive understanding of how everything fits together.

本教程将创建一个能够突出显示pastebin 代码的 Web API。在这个过程中，我们将介绍 REST framework 的各个组件，并让你对这些组件在一起是如何工作的有一个全面的了解。

The tutorial is fairly in-depth, so you should probably get a cookie and a cup of your favorite brew before getting started.  If you just want a quick overview, you should head over to the [quickstart] documentation instead.

本教程讲解的很全面，所以在开始前吃点你最爱的小点心补充一下能量吧。

---

**Note**: The code for this tutorial is available in the [tomchristie/rest-framework-tutorial][repo] repository on GitHub.  The completed implementation is also online as a sandbox version for testing, [available here][sandbox].

---

## Setting up a new environment

搭建一个新环境

Before we do anything else we'll create a new virtual environment, using [virtualenv].  This will make sure our package configuration is kept nicely isolated from any other projects we're working on.

在我们开始我的旅程之前，我们首先需要使用[virtualenv]来搭建一个虚拟环境。

这样就能确保我们的项目与你其他的项目之间彼此隔离。

    virtualenv env
    source env/bin/activate

Now that we're inside a virtualenv environment, we can install our package requirements.

    pip install django
    pip install djangorestframework
    pip install pygments  # We'll be using this for the code highlighting

**Note:** To exit the virtualenv environment at any time, just type `deactivate`.  For more information see the [virtualenv documentation][virtualenv].

只需要输入 `deactivate`就可以随时退出虚拟环境了。

## Getting started

Okay, we're ready to get coding.

好了，让我们开工吧。

To get started, let's create a new project to work with.

首先，创建一个新项目。

    cd ~
    django-admin.py startproject tutorial
    cd tutorial

Once that's done we can create an app that we'll use to create a simple Web API.

然后，我们在这个项目中创建一个应用，来完成我们的Web API 的构建。

    python manage.py startapp snippets
We'll need to add our new `snippets` app and the `rest_framework` app to `INSTALLED_APPS`. Let's edit the `tutorial/settings.py` file:

我们需要把 `snippets` 和`rest_framework`应用添加到`tutorial/settings.py`里的`INSTALLED_APPS`中。

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
)
```

Okay, we're ready to roll.

接下来，开始我们的旅程了。

## Creating a model to work with

For the purposes of this tutorial we're going to start by creating a simple `Snippet` model that is used to store code snippets.  Go ahead and edit the `snippets/models.py` file.  Note: Good programming practices include comments.  Although you will find them in our repository version of this tutorial code, we have omitted them here to focus on the code itself.

我们需要在`snippets/models.py` 中创建一个简单的 `Snippet` 模型来储存我们的代码片段。

注意：正常情况下，我们的代码里应该包含注释，用于告诉其他人与以后的自己，我们写的这些代码是干什么的，如何使用等。

```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())
```


```python
class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```

We'll also need to create an initial migration for our snippet model, and sync the database for the first time.

完成模型类后，就我们需要将它迁移到数据库中。

    python manage.py makemigrations snippets
    python manage.py migrate

## Creating a Serializer class

创建一个序列化类

The first thing we need to get started on our Web API is to provide a way of serializing and deserializing the snippet instances into representations such as `json`.  We can do this by declaring serializers that work very similar to Django's forms.  Create a file in the `snippets` directory named `serializers.py` and add the following.

首先，我们需要对 snippet 实例进行序列化和反序列化，（把snippet 实例序列化为json格式，从json格式反序列化为snippet 实例）。

我们可以建立一个和 Django 中的表单相似的序列化器来实现。在当前目录下创建 `serializers.py` 。

```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES
```


```python
class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```

The first part of the serializer class defines the fields that get serialized/deserialized.  The `create()` and `update()` methods define how fully fledged instances are created or modified when calling `serializer.save()`

serializer class 的第一部分定义了 进行序列化或者反序列化的字段。`create()` 和 `update()` 方法定义了在调用`serializer.save()`时如何创建或修改完整的实例。

A serializer class is very similar to a Django `Form` class, and includes similar validation flags on the various fields, such as `required`, `max_length` and `default`.

serializer class 与Django中的`Form` class非常相似，在各个字段具有相似的验证标志，比如`required`, `max_length` 和`default`.

The field flags can also control how the serializer should be displayed in certain circumstances, such as when rendering to HTML. The `{'base_template': 'textarea.html'}` flag above is equivalent to using `widget=widgets.Textarea` on a Django `Form` class. This is particularly useful for controlling how the browsable API should be displayed, as we'll see later in the tutorial.

字段标志还可以控制在某些情况下应该如何显示序列化程序，例如在呈现为HTML时。

We can actually also save ourselves some time by using the `ModelSerializer` class, as we'll see later, but for now we'll keep our serializer definition explicit.

实际上，我们也可以通过继承`ModelSerializer`类来节省一些时间，我们将在后面的代码中使用它，但是现在为了让大家对序列化器有更加详细的了解，我们将具体的过程清晰的展示给大家。

(通过上面的定义的简单序列化器，我们就可以完成snippet 实例序列化为json格式，从json格式反序列化为snippet 实例的操作了)

## Working with Serializers

让我们定义的序列化器工作起来

Before we go any further we'll familiarize ourselves with using our new Serializer class.  Let's drop into the Django shell.

先让我们看一看我们的定义的序列化器工作的怎么样，进入 Django shell。

    python manage.py shell
Okay, once we've got a few imports out of the way, let's create a couple of code snippets to work with.

接下来，让我们创建一些代码片段，在 Django shell 中输入下面的这些代码。

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print "hello, world"\n')
snippet.save()
```

We've now got a few snippet instances to play with.  Let's take a look at serializing one of those instances.

我们已经创建了几个 snippet 实例， 先序列化其中的一个snippet 实例。  

```python
serializer = SnippetSerializer(snippet)
serializer.data
# {'id': 2, 'title': u'', 'code': u'print "hello, world"\n', 'linenos': False, 'language': u'python', 'style': u'friendly'}
```

At this point we've translated the model instance into Python native datatypes.  To finalize the serialization process we render the data into `json`.

首先我们将 模型实例 转换为 Python数据类型。

接下来 ，我们将Python数据呈现为json格式，最终来完成整个序列化过程。

（注意看两步操作的结果，第一步操作的结果是 Python中的dict；第二步操作的结果是 json 类型。

```python
content = JSONRenderer().render(serializer.data)
content
# '{"id": 2, "title": "", "code": "print \\"hello, world\\"\\n", "linenos": false, "language": "python", "style": "friendly"}'
```

Deserialization is similar.  First we parse a stream into Python native datatypes...

反序列化与上面的序列化操作很相似。

首先，我们将数据流解析为Python数据类型（也就是把json类型转换为Python数据类型）。

```python
from django.utils.six import BytesIO
```

```python
stream = BytesIO(content)
data = JSONParser().parse(stream)
```

...then we restore those native datatypes into a fully populated object instance.

然后我们把Python数据类型还原成一个完整的实例。

```python
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
# <Snippet: Snippet object>
```

Notice how similar the API is to working with forms.  The similarity should become even more apparent when we start writing views that use our serializer.

你发现没有，我们写的API和forms 是多么的相似。待会我们写 views 那部分代码时，你会发现两者更加相似。

We can also serialize querysets instead of model instances.  To do so we simply add a `many=True` flag to the serializer arguments.

除了模型实例，我们还可以序列化查询集，我们只需要添加`many=True`参数就可以了。

```python
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
# [OrderedDict([('id', 1), ('title', u''), ('code', u'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', u''), ('code', u'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', u''), ('code', u'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
```

## Using ModelSerializers

使用 ModelSerializers

Our `SnippetSerializer` class is replicating a lot of information that's also contained in the `Snippet` model.  It would be nice if we could keep our code a bit  more concise.

`SnippetSerializer` class 中包含了大量与`Snippet` model 重复的内容，如果我们能够把这些重复的内容简化一下，那就更好了。

In the same way that Django provides both `Form` classes and `ModelForm` classes, REST framework includes both `Serializer` classes, and `ModelSerializer` classes.

在Django 中，除了`Form` classes 还提供了` ModelForm` classes来帮助我们简化代码，同样的，在 REST framework 中除了`Serializer` classes 外，还提供了`ModelSerializer`classes 来帮助我们简化代码。

Let's look at refactoring our serializer using the `ModelSerializer` class.

那么接下来，我们就要使用`ModelSerializer` class 来重构我们的 序列化器了。

Open the file `snippets/serializers.py` again, and replace the `SnippetSerializer` class with the following.

再次打开 `snippets/serializers.py` ，用下面的代码代替我们之前写的`SnippetSerializer`class。

```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```

One nice property that serializers have is that you can inspect all the fields in a serializer instance, by printing its representation. Open the Django shell with `python manage.py shell`, then try the following:

序列化器拥有一个非常不错的属性repr，我们可以通过打印 repr() 的结果来查看 serializer instance 中的所有字段。

你可一个在Django shell 中输入下面的代码，看一看结果是否与下面给出的结果一致。

```python
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...
```

It's important to remember that `ModelSerializer` classes don't do anything particularly magical, they are simply a shortcut for creating serializer classes:

* An automatically determined set of fields.
* Simple default implementations for the `create()` and `update()` methods.

我们要知道的是，`Modelserializer`classes 并没有做什么神奇的操作，它仅仅是简化了我们前面的操作，让我们不必在写很多重复的内容而已，他其实只是做了这俩个操作：

* 帮助我们自动定义了一些字段；
* 简单的实现了`create()`和`update()`方法。

## Writing regular Django views using our Serializer

使用序列化器完成常规的 Django views

Let's see how we can write some API views using our new Serializer class.

For the moment we won't use any of REST framework's other features, we'll just write the views as regular Django views.

接下来，让我们看一看如何使用我们刚刚创建的Serializer class 完成 API views。目前，我们不会使用REST framework的其他功能，我们仅仅把views编写为常规的Django views.



Edit the `snippets/views.py` file, and add the following.

在`snippets/views.py` 文件中，添加下面的代码。

```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```

The root of our API is going to be a view that supports listing all the existing snippets, or creating a new snippet.

我们编写的API view 具有列出存在的代码片段或者创建一个新的代码片段。

```python
@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

Note that because we want to be able to POST to this view from clients that won't have a CSRF token we need to mark the view as `csrf_exempt`.  This isn't something that you'd normally want to do, and REST framework views actually use more sensible behavior than this, but it'll do for our purposes right now.

我们想要客户端通过POST请求方式来向我们的view发起请求，但是我们没有 CSRF token ，所以我们需要使用`csrf_exempt`来装饰一下我们的view。

正常情况下，我们是不会使用这种方式来解决POST请求的，而且REST framework views 也有更好的方式来解决，但是现在为了演示方便，我们采用这种方式。

We'll also need a view which corresponds to an individual snippet, and can be used to retrieve, update or delete the snippet.

我们还需要编写一个view 来 retrieve、update、delete 代码片段。

```python
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

Finally we need to wire these views up.  Create the `snippets/urls.py` file:

编写完成上面的views后，新建`snippets/urls.py`文件来完成路由的配置：

```python
from django.conf.urls import url
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.snippet_detail),
]
```

We also need to wire up the root urlconf, in the `tutorial/urls.py` file, to include our snippet app's URLs.

我们需要在`tutorial/urls.py` 文件中配置根路由，将我们的snippet app的URLs 包含进去。

```python
from django.conf.urls import url, include
```

```python
urlpatterns = [
    url(r'^', include('snippets.urls')),
]
```

It's worth noting that there are a couple of edge cases we're not dealing with properly at the moment.  If we send malformed `json`, or if a request is made with a method that the view doesn't handle, then we'll end up with a 500 "server error" response.  Still, this'll do for now.

现在是时候检验一下我们的成果了，虽然还有一些小问题没处理好，比如：请求的`json`格式有问题、使用views没有包含的方法请求都会引发 500 服务器错误。但是这对我们要测试的功能并没有影响。

## Testing our first attempt at a Web API

Now we can start up a sample server that serves our snippets.

现在让我们来启动我们的服务。

Quit out of the shell...

先退出shell

	quit()
...and start up Django's development server.

然后启动Django自带的web server。

	python manage.py runserver

	Validating models...

	0 errors found
	Django version 1.11, using settings 'tutorial.settings'
	Development server is running at http://127.0.0.1:8000/
	Quit the server with CONTROL-C.

In another terminal window, we can test the server.

在另一个终端中，开始测试我们的服务。

We can test our API using [curl][curl] or [httpie][httpie]. Httpie is a user friendly http client that's written in Python. Let's install that.

我们也可以使用 [curl][curl] 或者 [httpie][httpie]来进行测试。Httpie 是使用Python编写的对用户非常友好的http 客户端，让我们安装一下它。

You can install httpie using pip:

使用pip安装

    pip install httpie
Finally, we can get a list of all of the snippets:

在终端中输入下面的请求，来查看所有的代码片段：

    http http://127.0.0.1:8000/snippets/

```json
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
```

Or we can get a particular snippet by referencing its id:

通过代码片段的id 来查看具体的某个代码片段：

    http http://127.0.0.1:8000/snippets/2/

```json
HTTP/1.1 200 OK
...
{
  "id": 2,
  "title": "",
  "code": "print \"hello, world\"\n",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}
```

Similarly, you can have the same json displayed by visiting these URLs in a web browser.

在浏览器中你输入与上面一样的网址，会得到同样的结果。

## Where are we now

We're doing okay so far, we've got a serialization API that feels pretty similar to Django's Forms API, and some regular Django views.

到现在，我们完成了一个和Django Forms API 相似的 serialization API，以及一些常规的Django views。

Our API views don't do anything particularly special at the moment, beyond serving `json` responses, and there are some error handling edge cases we'd still like to clean up, but it's a functioning Web API.

我们编写的API views 现在只能够服务`json`类型的响应，并且有一些小问题待解决，但是它是一个能够正常工作的 Web API。

We'll see how we can start to improve things in [part 2 of the tutorial][tut-2].

我们将从 [part 2 of the tutorial][tut-2]开始逐步改进我们的作品。

[quickstart]: quickstart.md
[repo]: https://github.com/encode/rest-framework-tutorial
[sandbox]: http://restframework.herokuapp.com/
[virtualenv]: http://www.virtualenv.org/en/latest/index.html
[tut-2]: 2-requests-and-responses.md
[httpie]: https://github.com/jakubroztocil/httpie#installation
[curl]: http://curl.haxx.se
[]: 
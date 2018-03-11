source: authentication.py

# Authentication

身份认证

> Auth needs to be pluggable.
>
> 身份验证功能应该是可插拔的。
>
> &mdash; Jacob Kaplan-Moss, ["REST worst practices"][cite]

Authentication is the mechanism of associating an incoming request with a set of identifying credentials, such as the user the request came from, or the token that it was signed with.  The [permission] and [throttling] policies can then use those credentials to determine if the request should be permitted.

身份验证是将传入的请求与一组识别凭证相关联的机制，而识别凭证可以是发送请求的用户或用户签名的令牌。

然后[权限]和[限制]策略可以使用这些凭据来确定是否允许请求。

（比如说一个用户以游客身份请求查询首页数据，我们允许该请求，并返回数据。但如果他请求修改数据，添加数据，[权限]策略发现游客身份不具备修改、添加数据的权限，[限制]策略不会允许该请求。而如果以管理员身份发送修改数据的请求，则会允许该请求）

REST framework provides a number of authentication schemes out of the box, and also allows you to implement custom schemes.

REST framework 提供了许多封装好的身份验证方案，但我们也可以根据自己的实际需求来自定义我们自己的身份验证方案。

Authentication is always run at the very start of the view, before the permission and throttling checks occur, and before any other code is allowed to proceed.

身份验证总是在view刚开始的时候执行，在执行权限与限制检查以及其他任何允许执行的代码之前。

The `request.user` property will typically be set to an instance of the `contrib.auth` package's `User` class.

`request.user`属性通常会被设置为`contrib.auth`包中`User`Class的一个实例。

The `request.auth` property is used for any additional authentication information, for example, it may be used to represent an authentication token that the request was signed with.

`request.auth`属性可以用于任何其他身份验证信息，例如，它可以用来表示某个请求已签名的身份令牌。

---

**Note:** Don't forget that **authentication by itself won't allow or disallow an incoming request**, it simply identifies the credentials that the request was made with.

注意：身份验证本身不允许传入请求，它只是标识请求的凭据。

（每个请求都带有标识自己身份的凭据，告诉API自己到底是游客身份还是管理员身份，然后根据身份来确定权限，最终决定是否允许该请求）

For information on how to setup the permission polices for your API please see the [permissions documentation][permission].

如果你想知道如何为你的API设置权限策略，请看[permissions documentation][permission].

---

## How authentication is determined

身份认证是如何确定的

The authentication schemes are always defined as a list of classes.  REST framework will attempt to authenticate with each class in the list, and will set `request.user` and `request.auth` using the return value of the first class that successfully authenticates.

认证方案是一个由很多classes组成的list。

REST framework将尝试使用list中的每个class进行认证，并将成功认证的第一个class的返回值来设置`request.user`和`request.auth`。

If no class authenticates, `request.user` will be set to an instance of `django.contrib.auth.models.AnonymousUser`, and `request.auth` will be set to `None`.

如果没有class可以用来进行身份验证，则将`request.user`设置为`django.contrib.auth.models.AnonymousUser`的实例，并将`request.auth`设置为`None`。

The value of `request.user` and `request.auth` for unauthenticated requests can be modified using the `UNAUTHENTICATED_USER` and `UNAUTHENTICATED_TOKEN` settings.

可以使用`UNAUTHENTICATED_USER`和`UNAUTHENTICATED_TOKEN`设置修改未经身份验证的请求的`request.user`和`request.auth`的值。

## Setting the authentication scheme

设置身份认证方案

The default authentication schemes may be set globally, using the `DEFAULT_AUTHENTICATION_CLASSES` setting.  For example.

我们可以将默认的身份认证方案设置为全局的，可以使用`DEFAULT_AUTHENTICATION_CLASSES`进行设置。例如：

```Python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    )
}
```

You can also set the authentication scheme on a per-view or per-viewset basis, using the `APIView` class-based views.

你还可以在每个view或每个viewset上面设置身份认证方案，下面是给基于`APIView`的view设置身份认证方案：

```Python
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    authentication_classes = (SessionAuthentication, BasicAuthentication)
    permission_classes = (IsAuthenticated,)

    def get(self, request, format=None):
        content = {
            'user': unicode(request.user),  # `django.contrib.auth.User` instance.
            'auth': unicode(request.auth),  # None
        }
        return Response(content)
```

Or, if you're using the `@api_view` decorator with function based views.

下面是给使用`@api_view`装饰器基于函数的view设置身份认证方案：

```Python
@api_view(['GET'])
@authentication_classes((SessionAuthentication, BasicAuthentication))
@permission_classes((IsAuthenticated,))
def example_view(request, format=None):
    content = {
        'user': unicode(request.user),  # `django.contrib.auth.User` instance.
        'auth': unicode(request.auth),  # None
    }
    return Response(content)
```

## Unauthorized and Forbidden responses

未通过的身份验证和拒绝请求的响应

When an unauthenticated request is denied permission there are two different error codes that may be appropriate.

当未经身份验证的请求被拒绝时，有两种不同的错误代码可能是比较合适的。

* [HTTP 401 Unauthorized][http401]
* [HTTP 403 Permission Denied][http403]

HTTP 401 responses must always include a `WWW-Authenticate` header, that instructs the client how to authenticate.  HTTP 403 responses do not include the `WWW-Authenticate` header.

HTTP 401响应必须始终包含`WWW-Authenticate`的header，该header指示客户端如何进行身份验证。

HTTP 403响应不包含`WWW-Authenticate`header。

The kind of response that will be used depends on the authentication scheme.  Although multiple authentication schemes may be in use, only one scheme may be used to determine the type of response.  **The first authentication class set on the view is used when determining the type of response**.

至于使用哪一种响应取决与身份认证方案。虽然我们可能会使用好几个身份认证方案，但是只有一个方案决定响应的类型。**view使用的第一个认证class决定response的类型**。

Note that when a request may successfully authenticate, but still be denied permission to perform the request, in which case a `403 Permission Denied` response will always be used, regardless of the authentication scheme.

请注意，当请求可以成功进行身份验证时，仍然可能会被拒绝执行请求，在这种情况下，将始终使用 `403 Permission Denied` response，身份验证方案将不会在决定response的类型。

## Apache mod_wsgi specific configuration

Apache mod_wsgi 的相关配置

Note that if deploying to [Apache using mod_wsgi][mod_wsgi_official], the authorization header is not passed through to a WSGI application by default, as it is assumed that authentication will be handled by Apache, rather than at an application level.

请注意，如果使用mod_wsgi部署到Apache，身份认证header在默认情况下不会传递到WSGI应用程序，因为它假定身份认证将由Apache处理，而不是由应用程序处理。

If you are deploying to Apache, and using any non-session based authentication, you will need to explicitly configure mod_wsgi to pass the required headers through to the application.  This can be done by specifying the `WSGIPassAuthorization` directive in the appropriate context and setting it to `'On'`.

如果您正在部署到Apache并使用任何基于非会话的身份验证，则需要明确配置mod_wsgi以将所需的headers传递给应用程序。

这可以通过在适当的上下文中指定`WSGIPassAuthorization`指令并将其设置为“On”来完成。

    # this can go in either server config, virtual host, directory or .htaccess
    # 这可以在服务器配置，虚拟主机，目录或.htaccess中进行
    WSGIPassAuthorization On

---

# API Reference

示例API

## BasicAuthentication

基础身份认证

This authentication scheme uses [HTTP Basic Authentication][basicauth], signed against a user's username and password.  Basic authentication is generally only appropriate for testing.

该认证方案使用[HTTP Basic Authentication][basicauth]，并根据用户的用户名和密码进行签名。 **基本认证通常只适用于测试**。

If successfully authenticated, `BasicAuthentication` provides the following credentials.

如果成功通过身份验证，`BasicAuthentication`将提供以下凭据。

* `request.user` will be a Django `User` instance.
* `request.user`是Django `User`class 的实例
* `request.auth` will be `None`.
* `request.auth` 将会是`None`

Unauthenticated responses that are denied permission will result in an `HTTP 401 Unauthorized` response with an appropriate WWW-Authenticate header.  For example:

如果未通过身份认证，将会因为权限不足而得到 `HTTP 401 Unauthorized`的response，

    WWW-Authenticate: Basic realm="api"
**Note:** If you use `BasicAuthentication` in production you must ensure that your API is only available over `https`.  You should also ensure that your API clients will always re-request the username and password at login, and will never store those details to persistent storage.

**注意**：如果您在生产环境中使用`BasicAuthentication`，则必须确保您的API仅可通过`https`访问（使用Http将会导致账号与密码的泄露）。 您还应该确保您的API客户端将始终在执行登录操作时重新输入用户名和密码进行请求，并且永远不会将这些详细信息存储到持久性存储中。

## TokenAuthentication

令牌身份认证

This authentication scheme uses a simple token-based HTTP Authentication scheme.  Token authentication is appropriate for client-server setups, such as native desktop and mobile clients.

此认证方案使用简单的基于令牌的HTTP认证方案。 令牌身份认证适用于client-server，例如本地桌面和移动客户端。

To use the `TokenAuthentication` scheme you'll need to [configure the authentication classes](#setting-the-authentication-scheme) to include `TokenAuthentication`, and additionally include `rest_framework.authtoken` in your `INSTALLED_APPS` setting:

要使用`TokenAuthentication`方案，您需要配置认证类以包含`TokenAuthentication`，并且还需要在`INSTALLED_APPS`设置中包含`rest_framework.authtoken`：

```Python
INSTALLED_APPS = (
    ...
    'rest_framework.authtoken'
)
```

---

**Note:** Make sure to run `manage.py migrate` after changing your settings. The `rest_framework.authtoken` app provides Django database migrations.

在配置完setting后，需要迁移数据库，执行`manage.py  migrate`。

---

You'll also need to create tokens for your users.

你需要为你的users创建令牌。

```Python
from rest_framework.authtoken.models import Token
```

```Python
token = Token.objects.create(user=...)
print token.key
```

For clients to authenticate, the token key should be included in the `Authorization` HTTP header.  The key should be prefixed by the string literal "Token", with whitespace separating the two strings.  For example:

客户端进行身份验证时，令牌密钥应包含在`Authorization`HTTP header中。 密匙应以字符串文字“Token”为前缀，用空格分隔两个字符串。 例如：

    Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
**Note:** If you want to use a different keyword in the header, such as `Bearer`, simply subclass `TokenAuthentication` and set the `keyword` class variable.

如果您想在header中使用不同的关键字（例如`Bearer`），只需设置`keyword`class为`TokenAuthentication`的子类。

If successfully authenticated, `TokenAuthentication` provides the following credentials.

如果身份认证成功，`TokenAuthentication`提供了下列凭据：

* `request.user` will be a Django `User` instance.
* `request.user 将是Django `User` class 的实例。
* `request.auth` will be a `rest_framework.authtoken.models.Token` instance.
* request.auth 将是 `rest_framework.authtoken.models.Token`的实例。

Unauthenticated responses that are denied permission will result in an `HTTP 401 Unauthorized` response with an appropriate WWW-Authenticate header.  For example:

如果未通过身份认证，将会因为权限不足而得到 `HTTP 401 Unauthorized`的response

    WWW-Authenticate: Token
The `curl` command line tool may be useful for testing token authenticated APIs.  For example:

你可以用`curl`命令行工具测试你的 token authenticated APIs。

    curl -X GET http://127.0.0.1:8000/api/example/ -H 'Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b'
---

**Note:** If you use `TokenAuthentication` in production you must ensure that your API is only available over `https`.

如果您在生产中使用TokenAuthentication，则必须确保您的API只能通过https访问。

---

#### Generating Tokens

生成令牌

##### By using signals

使用签名

If you want every user to have an automatically generated Token, you can simply catch the User's `post_save` signal.

如果您希望每个用户都拥有一个自动生成的令牌，则只需捕捉用户的`post_save`签名即可。

```Python
from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

@receiver(post_save, sender=settings.AUTH_USER_MODEL)
def create_auth_token(sender, instance=None, created=False, **kwargs):
    if created:
        Token.objects.create(user=instance)
```

Note that you'll want to ensure you place this code snippet in an installed `models.py` module, or some other location that will be imported by Django on startup.

请注意，您需要确保将此代码片段放置在已安装的`models.py`模块或Django启动时将导入的其他某个位置。

If you've already created some users, you can generate tokens for all existing users like this:

如果您已经创建了一些用户，则可以为所有现有用户生成令牌，例如：

```Python
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

for user in User.objects.all():
    Token.objects.get_or_create(user=user)
```

##### By exposing an api endpoint

预留API接口

When using `TokenAuthentication`, you may want to provide a mechanism for clients to obtain a token given the username and password.  REST framework provides a built-in view to provide this behavior.  To use it, add the `obtain_auth_token` view to your URLconf:

使用`TokenAuthentication`时，您可能希望为客户端提供一种机制，以获取给定`username`和`password`的令牌。

REST framework提供了一个内置的view来提供这种行为。 如果要使用它，请将`obtain_auth_token`view添加到您的URLconf中：

```Python
from rest_framework.authtoken import views
urlpatterns += [
    url(r'^api-token-auth/', views.obtain_auth_token)
]
```

Note that the URL part of the pattern can be whatever you want to use.

URL模式的URL部分，你可以自己决定。

The `obtain_auth_token` view will return a JSON response when valid `username` and `password` fields are POSTed to the view using form data or JSON:

当使用表单数据或JSON将有效的`username`和`password`字段POSTed到view时，`obtain_auth_token`view将返回JSON响应：

```json
{ 'token' : '9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b' }
```
Note that the default `obtain_auth_token` view explicitly uses JSON requests and responses, rather than using default renderer and parser classes in your settings.  If you need a customized version of the `obtain_auth_token` view, you can do so by overriding the `ObtainAuthToken` view class, and using that in your url conf instead.

请注意，默认的`obtain_auth_token`view显式使用JSONj进行请求和响应，而不是使用你在setting中配置的默认的渲染器和分析器类。如果您需要自定义版的`obtain_auth_token`view，您可以通过覆盖`ObtainAuthToken`view class，并在您的url conf中使用它来代替。

By default there are no permissions or throttling applied to the  `obtain_auth_token` view. If you do wish to apply throttling you'll need to override the view class, and include them using the `throttle_classes` attribute.

默认情况下，`obtain_auth_token`没有添加权限控制或者限制。如果你希望添加权限控制，那你需要重写view class，为view class 添加`throttle_classes`属性，将你需要的权限控制添加到该属性。

##### With Django admin

使用Django admin

It is also possible to create Tokens manually through admin interface. In case you are using a large user base, we recommend that you monkey patch the `TokenAdmin` class to customize it to your needs, more specifically by declaring the `user` field as `raw_field`.

我们也可以通过Django 后台管理界面手动创建令牌。如果用户群很大，我们建议您对`TokenAdmin`类进行修补以根据需要对其进行定制，更具体地说，将`user`field声明为`raw_field`。

`your_app/admin.py`:

```Python
from rest_framework.authtoken.admin import TokenAdmin
```

```Python
TokenAdmin.raw_id_fields = ('user',)
```


#### Using Django manage.py command

使用Django manage.py 命令

Since version 3.6.4 it's possible to generate a user token using the following command:

从3.6.4开始我们可以下列命令生成一个用户令牌。

    ./manage.py drf_create_token <username>
this command will return the API token for the given user, creating it if it doesn't exist:

这个命令将会返回一个与你传入的user相对应的API令牌。

    Generated token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b for user user1
In case you want to regenerate the token (for example if it has been compromised or leaked) you can pass an additional parameter:

如果您想重新生成令牌（比如说，之前的令牌已被泄漏），则可以传递一个附加参数‘-r'：

    ./manage.py drf_create_token -r <username>


## SessionAuthentication

会话身份认证

This authentication scheme uses Django's default session backend for authentication.  Session authentication is appropriate for AJAX clients that are running in the same session context as your website.

此认证方案使用Django的默认会话后端进行认证。会话身份认证适用于与您的网站在同一会话环境中运行的AJAX客户端。

If successfully authenticated, `SessionAuthentication` provides the following credentials.

如果身份认证成功，`SessionAuthentication`提供下列凭证。

* `request.user` will be a Django `User` instance.
* `request.user`将是Django `User`class 的实例。
* `request.auth` will be `None`.
* `request.auth`将为`None`。

Unauthenticated responses that are denied permission will result in an `HTTP 403 Forbidden` response.

如果未通过身份认证，将会得到`HTTP 403 Forbidden`的响应。

If you're using an AJAX style API with SessionAuthentication, you'll need to make sure you include a valid CSRF token for any "unsafe" HTTP method calls, such as `PUT`, `PATCH`, `POST` or `DELETE` requests.  See the [Django CSRF documentation][csrf-ajax] for more details.

如果你使用的AJAX风格的API，那么为了通过身份认证，你需要为所有请求的“不安全”HTTP方法（比如`put`、`POST`、`DELETE`、`PATCH`）添加一个有效的CSRF token。如果你想了解更多的细节请参考[Django CSRF documentation][csrf-ajax] 。

**Warning**: Always use Django's standard login view when creating login pages. This will ensure your login views are properly protected.

**警告**：如果你想创建一个登录页面，那你应该使用Django 标准的 login view。这样可以确保你的 login view 得到适当的保护。

CSRF validation in REST framework works slightly differently to standard Django due to the need to support both session and non-session based authentication to the same views. This means that only authenticated requests require CSRF tokens, and anonymous requests may be sent without CSRF tokens. This behaviour is not suitable for login views, which should always have CSRF validation applied.

CSRF 验证在 REST framework 与 Django 中并不一样，在REST framework中CSRF 验证需要支持基于会话与非会话的身份认证。这意味着只有经过身份验证的请求才需要CSRF令牌，并且可以在没有CSRF令牌的情况下发送匿名请求。此行为不适用于应始终应用CSRF验证的登录视图。

## RemoteUserAuthentication

远程用户身份认证

This authentication scheme allows you to delegate authentication to your web server, which sets the `REMOTE_USER`
environment variable.

这种身份验证方案允许您将身份验证委托给您的Web服务器，该服务器需要设置`REMOTE_USER`环境变量。

To use it, you must have `django.contrib.auth.backends.RemoteUserBackend` (or a subclass) in your
`AUTHENTICATION_BACKENDS` setting. By default, `RemoteUserBackend` creates `User` objects for usernames that don't
already exist. To change this and other behaviour, consult the Django documentation](https://docs.djangoproject.com/en/stable/howto/auth-remote-user/).

要使用远程身份认证，你必须在你的`AUTHENTICATION_BACKENDS`设置中有`django.contrib.auth.backends.RemoteUserBackend`（或者`RemoteUserBackend`的一个子类）。

默认情况下，`RemoteUserBackend`为不存在的用户名创建用户对象。

如果你想要自定义`RemoteUserbackend`的行为，请参考Django文档(https://docs.djangoproject.com/en/stable/howto/auth-remote-user/).。

If successfully authenticated, `RemoteUserAuthentication` provides the following credentials:

如果身份认证成功，`RemoteUserAuthentication`提供下列凭据:

* `request.user` will be a Django `User` instance.
* `request.user`将是 一个Django `User` class 实例。
* `request.auth` will be `None`.
* `request.auth`将是`None`。

Consult your web server's documentation for information about configuring an authentication method, e.g.:

有关配置身份验证方法的信息，请查阅您的Web服务器的文档:

* [Apache Authentication How-To](https://httpd.apache.org/docs/2.4/howto/auth.html)
* [NGINX (Restricting Access)](https://www.nginx.com/resources/admin-guide/#restricting_access)


# Custom authentication

自定义身份认证

To implement a custom authentication scheme, subclass `BaseAuthentication` and override the `.authenticate(self, request)` method.  The method should return a two-tuple of `(user, auth)` if authentication succeeds, or `None` otherwise.

要实现自定义身份验证方案，请创建`BaseAuthentication`的子类，并重写`.authenticate(self，request)`方法。

In some circumstances instead of returning `None`, you may want to raise an `AuthenticationFailed` exception from the `.authenticate()` method。如果认证成功，该方法应返回`(user，auth)`的二元元组，否则返回`None`。

Typically the approach you should take is:

通常你应该这样做：

* If authentication is not attempted, return `None`.  Any other authentication schemes also in use will still be checked.
* 如果没有尝试使用我们自定义的认证方案进行认证，则返回`None`。那么会尝试使用其他的身份认证方案进行认证。
* If authentication is attempted but fails, raise a `AuthenticationFailed` exception.  An error response will be returned immediately, regardless of any permissions checks, and without checking any other authentication schemes.
* 如果使用我们自定义的认证方案进行了认证，但是认证失败，抛出`AuthenticationFailed`异常。并且不再进行权限检查，不再使用其他的认证方案进行认证，而是立即返回一个错误响应。

You *may* also override the `.authenticate_header(self, request)` method.  If implemented, it should return a string that will be used as the value of the `WWW-Authenticate` header in a `HTTP 401 Unauthorized` response.

你也可以重写`.authenticate_header(self, request)`方法。它应该返回一个字符串，该字符串将用作`HTTP 401 Unauthorized`响应中的`WWW-Authenticate`header的值。

If the `.authenticate_header()` method is not overridden, the authentication scheme will return `HTTP 403 Forbidden` responses when an unauthenticated request is denied access.

如果没有重写`.authenticate_header()`方法，一个未经身份认证的请求被拒绝访问时，将会返回`HTTP 403 Forbidden`的响应。

---

**Note:** When your custom authenticator is invoked by the request object's `.user` or `.auth` properties, you may see an `AttributeError` re-raised as a `WrappedAttributeError`. This is necessary to prevent the original exception from being suppressed by the outer property access. Python will not recognize that the `AttributeError` orginates from your custom authenticator and will instead assume that the request object does not have a `.user` or `.auth` property. These errors should be fixed or otherwise handled by your authenticator.

当请求对象的`.user`或`.auth`属性调用您的自定义身份验证器时，您可能会看到`AttributeError`作为`WrappedAttributeError`被重新抛出。这对于防止原始异常被外部属性访问所抑制是必要的。Python不会识别来自你自定义的认证器的`AttributeError`，而是会假设request 对象没有`.user`或者`.auth`属性。这些错误应该由你定义的认证器来修复或处理。

---

## Example

The following example will authenticate any incoming request as the user given by the username in a custom request header named 'X_USERNAME'.

在下面的例子中：我们先通过一个名为‘X_USERNAME’的request  header来获取‘username’，然后我们通过‘username’找到对应的‘user’，我们将使用‘user’的身份来认证请求。

```Python
from django.contrib.auth.models import User
from rest_framework import authentication
from rest_framework import exceptions

class ExampleAuthentication(authentication.BaseAuthentication):
    def authenticate(self, request):
        username = request.META.get('X_USERNAME')
        if not username:
            return None

        try:
            user = User.objects.get(username=username)
        except User.DoesNotExist:
            raise exceptions.AuthenticationFailed('No such user')

        return (user, None)
```

---

# Third party packages

第三方包

The following third party packages are also available.

我们也可以使用以下的第三方包来帮助我们完成身份认证的功能。

## Django OAuth Toolkit

The [Django OAuth Toolkit][django-oauth-toolkit] package provides OAuth 2.0 support, and works with Python 2.7 and Python 3.3+. The package is maintained by [Evonove][evonove] and uses the excellent [OAuthLib][oauthlib].  The package is well documented, and well supported and is currently our **recommended package for OAuth 2.0 support**.

Django OAuth Toolkit包提供OAuth 2.0支持，支持的Python版本为Python 2.7和Python 3.3+。

该软件包由Evonove维护并使用优秀的OAuthLib。

该软件包有详细的文档，并得到了很好的支持，目前是我们推荐的OAuth 2.0支持软件包。

#### Installation & configuration

安装和配置

Install using `pip`.

使用 pip 安装

    pip install django-oauth-toolkit
Add the package to your `INSTALLED_APPS` and modify your REST framework settings.

将包添加到`INSTALLED_APPS`中并且修改REST framework 的配置。

```Python
INSTALLED_APPS = (
    ...
    'oauth2_provider',
)

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'oauth2_provider.contrib.rest_framework.OAuth2Authentication',
    )
}
```

For more details see the [Django REST framework - Getting started][django-oauth-toolkit-getting-started] documentation.

跟过详细信息请查阅[Django REST framework - Getting started][django-oauth-toolkit-getting-started]文档。

## Django REST framework OAuth

The [Django REST framework OAuth][django-rest-framework-oauth] package provides both OAuth1 and OAuth2 support for REST framework.

Django REST framework OAuth包为REST framework提供了OAuth1和OAuth2支持。

This package was previously included directly in REST framework but is now supported and maintained as a third party package.

此软件包之前直接包含在REST framework中，但现在作为第三方软件包得到支持和维护。

#### Installation & configuration

安装和配置

Install the package using `pip`.

使用 pip 安装

    pip install djangorestframework-oauth
For details on configuration and usage see the Django REST framework OAuth documentation for [authentication][django-rest-framework-oauth-authentication] and [permissions][django-rest-framework-oauth-permissions].

想了解更多配置的详细信息和身份认证与权限设置的用法请查阅 Django OAuth documentation。

## Digest Authentication

摘要认证

HTTP digest authentication is a widely implemented scheme that was intended to replace HTTP basic authentication, and which provides a simple encrypted authentication mechanism. [Juan Riaza][juanriaza] maintains the [djangorestframework-digestauth][djangorestframework-digestauth] package which provides HTTP digest authentication support for REST framework.

HTTP摘要认证是一种广泛实施的方案，旨在取代HTTP基本认证，并提供简单的加密认证机制。

Juan Riaza维护`djangorestframework-digestauth`包，为REST framework提供HTTP摘要认证支持。

（目前，`djangorestframework-digestauth`包已经停止维护，请使用者注意）

## Django OAuth2 Consumer

The [Django OAuth2 Consumer][doac] library from [Rediker Software][rediker] is another package that provides [OAuth 2.0 support for REST framework][doac-rest-framework].  The package includes token scoping permissions on tokens, which allows finer-grained access to your API.

来自Rediker Software的Django OAuth2 Consumer库是另一个为REST framework提供OAuth 2.0支持的软件包。

该软件包在令牌控制上包含令牌作用域权限，这允许对API进行更细粒度的访问。

## JSON Web Token Authentication

JSON Web Token is a fairly new standard which can be used for token-based authentication. Unlike the built-in TokenAuthentication scheme, JWT Authentication doesn't need to use a database to validate a token. [Blimp][blimp] maintains the [djangorestframework-jwt][djangorestframework-jwt] package which provides a JWT Authentication class as well as a mechanism for clients to obtain a JWT given the username and password. An alternative package for JWT authentication is [djangorestframework-simplejwt][djangorestframework-simplejwt] which provides different features as well as a pluggable token blacklist app.

JSON Web Token是一种相当新的标准，可用于基于令牌的身份验证。与内置的TokenAuthentication方案不同，JWT身份验证不需要使用数据库来验证令牌。Blimp维护djangorestframework-jwt软件包，该软件包提供JWT身份验证类以及客户机根据用户名和密码获取JWT的机制。JWT认证的另一个软件包是djangorestframework-simplejwt，它提供了不同的功能以及可插入的令牌黑名单应用程序。

## Hawk HTTP Authentication

The [HawkREST][hawkrest] library builds on the [Mohawk][mohawk] library to let you work with [Hawk][hawk] signed requests and responses in your API. [Hawk][hawk] lets two parties securely communicate with each other using messages signed by a shared key. It is based on [HTTP MAC access authentication][mac] (which was based on parts of [OAuth 1.0][oauth-1.0a]).

HawkREST库建立在Mohawk库上，让您可以在API中使用Hawk签名的请求和响应。Hawk让双方使用共享密钥签名的消息安全地进行通信。它基于 [HTTP MAC access authentication][mac]（基于OAuth 1.0的一部分）。

## HTTP Signature Authentication

HTTP Signature (currently a [IETF draft][http-signature-ietf-draft]) provides a way to achieve origin authentication and message integrity for HTTP messages. Similar to [Amazon's HTTP Signature scheme][amazon-http-signature], used by many of its services, it permits stateless, per-request authentication. [Elvio Toccalino][etoccalino] maintains the [djangorestframework-httpsignature][djangorestframework-httpsignature] package which provides an easy to use HTTP Signature Authentication mechanism.

HTTP签名（目前是IETF草案）提供了一种实现HTTP消息的原始认证和消息完整性的方法。与亚马逊的许多服务所使用的HTTP签名方案类似，它允许每个无状态的请求进行身份验证。Elvio Toccalino维护djangorestframework-httpsignature包，该包提供了一个易于使用的HTTP签名认证机制。

## Djoser

[Djoser][djoser] library provides a set of views to handle basic actions such as registration, login, logout, password reset and account activation. The package works with a custom user model and it uses token based authentication. This is a ready to use REST implementation of Django authentication system.

Djoser库提供了一系列视图来处理注册，登录，注销，密码重置和帐户激活等基本操作。该包使用自定义的user 模型并且使用基于令牌的身份认证。这是一个可以使用Django身份验证系统的REST实现。

## django-rest-auth

[Django-rest-auth][django-rest-auth] library provides a set of REST API endpoints for registration, authentication (including social media authentication), password reset, retrieve and update user details, etc. By having these API endpoints, your client apps such as AngularJS, iOS, Android, and others can communicate to your Django backend site independently via REST APIs for user management.

Django-rest-auth库提供了一组用于注册，认证（包括社交媒体认证），密码重置，检索和更新用户详细信息等的REST API接口。通过使用这些API接口，您的客户端应用程序（例如AngularJS，iOS，Android等）可以通过REST API独立地与您的Django后端站点通信以进行用户管理。

## django-rest-framework-social-oauth2

[Django-rest-framework-social-oauth2][django-rest-framework-social-oauth2] library provides an easy way to integrate social plugins (facebook, twitter, google, etc.) to your authentication system and an easy oauth2 setup. With this library, you will be able to authenticate users based on external tokens (e.g. facebook access token), convert these tokens to "in-house" oauth2 tokens and use and generate oauth2 tokens to authenticate your users.

Django-rest-framework-social-oauth2库提供了一种将社交插件（facebook，twitter，google等）集成到您的身份验证系统和简单的oauth2设置的简单方法。借助此库，您将能够基于外部令牌（例如Facebook访问令牌）对用户进行身份验证，将这些令牌转换为“内部”oauth2令牌，使用并生成oauth2令牌来验证您的用户。

## django-rest-knox

[Django-rest-knox][django-rest-knox] library provides models and views to handle token based authentication in a more secure and extensible way than the built-in TokenAuthentication scheme - with Single Page Applications and Mobile clients in mind. It provides per-client tokens, and views to generate them when provided some other authentication (usually basic authentication), to delete the token (providing a server enforced logout) and to delete all tokens (logs out all clients that a user is logged into).

考虑到单页应用程序和移动客户端的安全性和可扩展性。Django-rest-knox库提供了models和views来处理基于令牌的身份验证，而不是内置TokenAuthentication方案 。它提供了每个客户端的令牌以及一些views。在进行其他身份验证（通常是基本身份验证）时生成这些令牌的视图，执行删除令牌（提供服务器强制注销）和删除所有令牌（注销用户登录到的所有客户端）的views。

## drfpasswordless

[drfpasswordless][drfpasswordless] adds (Medium, Square Cash inspired) passwordless support to Django REST Framework's own TokenAuthentication scheme. Users log in and sign up with a token sent to a contact point like an email address or a mobile number.

drfpasswordless增加了对Django REST Framework 中的 Token Authentication方案的无密码支持（Medium，Square Cash的启发）。在用户注册和登录时将会把令牌发送给联系点（在注册时预留的联系方式）比如说：邮箱地址和手机号码。

[cite]: http://jacobian.org/writing/rest-worst-practices/
[http401]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2
[http403]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.4
[basicauth]: http://tools.ietf.org/html/rfc2617
[oauth]: http://oauth.net/2/
[permission]: permissions.md
[throttling]: throttling.md
[csrf-ajax]: https://docs.djangoproject.com/en/stable/ref/csrf/#ajax
[mod_wsgi_official]: http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIPassAuthorization
[django-oauth-toolkit-getting-started]: https://django-oauth-toolkit.readthedocs.io/en/latest/rest-framework/getting_started.html
[django-rest-framework-oauth]: http://jpadilla.github.io/django-rest-framework-oauth/
[django-rest-framework-oauth-authentication]: http://jpadilla.github.io/django-rest-framework-oauth/authentication/
[django-rest-framework-oauth-permissions]: http://jpadilla.github.io/django-rest-framework-oauth/permissions/
[juanriaza]: https://github.com/juanriaza
[djangorestframework-digestauth]: https://github.com/juanriaza/django-rest-framework-digestauth
[oauth-1.0a]: http://oauth.net/core/1.0a
[django-oauth-plus]: http://code.larlet.fr/django-oauth-plus
[django-oauth2-provider]: https://github.com/caffeinehit/django-oauth2-provider
[django-oauth2-provider-docs]: https://django-oauth2-provider.readthedocs.io/en/latest/
[rfc6749]: http://tools.ietf.org/html/rfc6749
[django-oauth-toolkit]: https://github.com/evonove/django-oauth-toolkit
[evonove]: https://github.com/evonove/
[oauthlib]: https://github.com/idan/oauthlib
[doac]: https://github.com/Rediker-Software/doac
[rediker]: https://github.com/Rediker-Software
[doac-rest-framework]: https://github.com/Rediker-Software/doac/blob/master/docs/integrations.md#
[blimp]: https://github.com/GetBlimp
[djangorestframework-jwt]: https://github.com/GetBlimp/django-rest-framework-jwt
[djangorestframework-simplejwt]: https://github.com/davesque/django-rest-framework-simplejwt
[etoccalino]: https://github.com/etoccalino/
[djangorestframework-httpsignature]: https://github.com/etoccalino/django-rest-framework-httpsignature
[amazon-http-signature]: http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html
[http-signature-ietf-draft]: https://datatracker.ietf.org/doc/draft-cavage-http-signatures/
[hawkrest]: https://hawkrest.readthedocs.io/en/latest/
[hawk]: https://github.com/hueniverse/hawk
[mohawk]: https://mohawk.readthedocs.io/en/latest/
[mac]: http://tools.ietf.org/html/draft-hammer-oauth-v2-mac-token-05
[djoser]: https://github.com/sunscrapers/djoser
[django-rest-auth]: https://github.com/Tivix/django-rest-auth
[django-rest-framework-social-oauth2]: https://github.com/PhilipGarnero/django-rest-framework-social-oauth2
[django-rest-knox]: https://github.com/James1345/django-rest-knox
[drfpasswordless]: https://github.com/aaronn/django-rest-framework-passwordless

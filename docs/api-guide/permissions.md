source: permissions.py

源码文件为permission.py

# Permissions

权限

> Authentication or identification by itself is not usually sufficient to gain access to information or code.  For that, the entity requesting access must have authorization.
>
> 仅仅通过身份验证或识别通常不足以允许请求获取信息或代码。 为此，请求访问的实体必须具有一定的授权才可以。
>
> &mdash; [Apple Developer Documentation][cite]

Together with [authentication] and [throttling], permissions determine whether a request should be granted or denied access.

`authentication`、`throttling`、`permission`一起来确定允许请求还是拒绝请求。

Permission checks are always run at the very start of the view, before any other code is allowed to proceed.  Permission checks will typically use the authentication information in the `request.user` and `request.auth` properties to determine if the incoming request should be permitted.

权限检查操作应该在view开始的时候进行，然后在执行其他的代码。权限检查通常会使用`request.user`和`request.auth`属性中的认证信息来确定是否允许传入的请求。

Permissions are used to grant or deny access different classes of users to different parts of the API.

权限用于授权或拒绝不同的用户访问API的不同部分。

The simplest style of permission would be to allow access to any authenticated user, and deny access to any unauthenticated user. This corresponds the `IsAuthenticated` class in REST framework.

最简单例子，允许任何经过身份验证的用户的访问，并拒绝任何未经身份验证的用户的访问。REST framework 中的`IsAuthenticated`class 就提供了这样的基本功能。

A slightly less strict style of permission would be to allow full access to authenticated users, but allow read-only access to unauthenticated users. This corresponds to the `IsAuthenticatedOrReadOnly` class in REST framework.

再举一个稍微区分不那么严格的例子，经过身份验证的用户具有全部的访问权限，但未经身份验证的用户仅有只读访问的权限。

## How permissions are determined

如何定义权限

Permissions in REST framework are always defined as a list of permission classes.

在REST framework中，我们总是把permission classes 定义到一个 list 中。

Before running the main body of the view each permission in the list is checked.

在开始执行view时，我们先检查 list 中的permission classes，以确定接收到的请求是否具有权限。

If any permission check fails an `exceptions.PermissionDenied` or `exceptions.NotAuthenticated` exception will be raised, and the main body of the view will not run.

一旦在检查权限失败，则会抛出`exceptions.PermissionDenied`或者`exception.NotAuthenticated`的异常，view 中的代码将不在执行。

When the permissions checks fail either a "403 Forbidden" or a "401 Unauthorized" response will be returned, according to the following rules:

当权限检查失败时，根据以下规则，将返回“403 Forbidden”或“401 Unauthorized”响应：

* The request was successfully authenticated, but permission was denied. *&mdash; An HTTP 403 Forbidden response will be returned.*
* 请求经过了身份认证，但是权限被拒绝。这时将会返回`403 Forbidden`的响应。
* The request was not successfully authenticated, and the highest priority authentication class *does not* use `WWW-Authenticate` headers. *&mdash; An HTTP 403 Forbidden response will be returned.*
* 该请求未成功通过身份验证，并且最高优先级的身份验证类未使用`WWW-Authenticate`headers。那么也将返回`403 Forbidden`的响应。
* The request was not successfully authenticated, and the highest priority authentication class *does* use `WWW-Authenticate` headers. *&mdash; An HTTP 401 Unauthorized response, with an appropriate `WWW-Authenticate` header will be returned.*
* 请求没有成功的通过身份验证，但是最高级的身份认证类使用了`WWW-Authenticate `headers。那么将返回`401 Unauthorized`的响应。

## Object level permissions

对象水平的权限

REST framework permissions also support object-level permissioning.  Object level permissions are used to determine if a user should be allowed to act on a particular object, which will typically be a model instance.

REST framework 支持设置对象级别的权限。对象级权限用于确定是否允许用户对特定对象执行操作，该特定对象通常是模型实例。

Object level permissions are run by REST framework's generic views when `.get_object()` is called.
As with view level permissions, an `exceptions.PermissionDenied` exception will be raised if the user is not allowed to act on the given object.

`.get_object()`被调用时，对象级权限由REST framework 的 generic views执行。

与视图级权限一样，如果用户不被允许对给定对象进行操作，则会引发`exceptions.PermissionDenied`的异常。

If you're writing your own views and want to enforce object level permissions,
or if you override the `get_object` method on a generic view, then you'll need to explicitly call the `.check_object_permissions(request, obj)` method on the view at the point at which you've retrieved the object.

如果您正在编写自己的视图并希望强制执行对象级权限，或者你重写了generic view 的`get_object`方法，那么您需要在视图里显式调用`.check_object_permissions(request，obj)`方法来指向你要检索的对象。

This will either raise a `PermissionDenied` or `NotAuthenticated` exception, or simply return if the view has the appropriate permissions.

如果请求没有权限，将引发`PermissionDenied`或`NotAuthenticated`异常，只有在请求拥有权限时才返回`obj`。

For example:

```python
def get_object(self):
    obj = get_object_or_404(self.get_queryset(), pk=self.kwargs["pk"])
    self.check_object_permissions(self.request, obj)
    return obj
```

#### Limitations of object level permissions

对象级别权限的局限性

For performance reasons the generic views will not automatically apply object level permissions to each instance in a queryset when returning a list of objects.

出于性能的考虑，generic views在返回对象列表时不会自动将对象级权限应用于查询集中的每个实例。

Often when you're using object level permissions you'll also want to [filter the queryset][filtering] appropriately, to ensure that users only have visibility onto instances that they are permitted to view.

一般情况下，当您使用对象级权限时，您还需要适当地过滤查询集，以确保用户只能看到他们被允许查看的实例。

## Setting the permission policy

在 settings.py 中配置权限

The default permission policy may be set globally, using the `DEFAULT_PERMISSION_CLASSES` setting. 

默认的权限设置将会全局有效，我们在`DEFAULT_PERMISSION_CLASSES`中进行配置。

 For example.

```Python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    )
}
```

If not specified, this setting defaults to allowing unrestricted access:

如果进行权限配置，那么默认情况下是无限制的访问权限（任何请求具有全部的访问权限）。

```Python
'DEFAULT_PERMISSION_CLASSES': (
   'rest_framework.permissions.AllowAny',
)
```

You can also set the permission policy on a per-view, or per-viewset basis, using the `APIView` class-based views.

你可以在基于类的每个view或者viewset上面设置权限策略。

```Python
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    permission_classes = (IsAuthenticated,)

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)
```

Or, if you're using the `@api_view` decorator with function based views.

当然，你也可以在使用`@api_view`装饰器进行装饰的（基于函数的）view上面配置权限策略。

```Python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

@api_view(['GET'])
@permission_classes((IsAuthenticated, ))
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```

__Note:__ when you set new permission classes through class attribute or decorators you're telling the view to ignore the default list set over the __settings.py__ file.

当您通过类属性或装饰器为view设置新的权限策略时，意味着您在告诉view忽略__settings.py__文件中设置的默认权限策略。

（你不在view里设置权限策略时，那么view将会使用`settings.py`里配置的 `DEFAULT_PERMISSION_CLASSES`；

而当你通过类属性或装饰器为view设置新的权限策略时，那么新的权限策略将会生效，而`DEFAULT_PERMISSION_CLASSES`的配置将不会生效。）

---

# API Reference

API 参考

## AllowAny

无限制的访问权限

The `AllowAny` permission class will allow unrestricted access, **regardless of if the request was authenticated or unauthenticated**.

`AllowAny`权限类将允许不受限制的访问，**不管该请求是通过身份验证还是未经身份验证**。

This permission is not strictly required, since you can achieve the same result by using an empty list or tuple for the permissions setting, but you may find it useful to specify this class because it makes the intention explicit.

并不是要求你在配置无限制访问权限时必须使用`AlllowAny` class，因为您可以通过为权限设置使用空列表或元组来获得相同的结果，但是您可能会发现指定此类是有用的，因为它会使意图变得明确。

## IsAuthenticated

经过身份验证的权限

The `IsAuthenticated` permission class will deny permission to any unauthenticated user, and allow permission otherwise.

`IsAuthenticated`权限类将拒绝任何未经身份验证的用户的任何权限，而通过身份验证的用户则拥有所有的权限。

This permission is suitable if you want your API to only be accessible to registered users.

如果您希望您的API只能由注册用户访问，则此权限很适合你。

## IsAdminUser

管理员权限

The `IsAdminUser` permission class will deny permission to any user, unless `user.is_staff` is `True` in which case permission will be allowed.

`IsAdminUser`权限类将拒绝任何用户的请求，除非`user.is_staff`为True（那么这个用户是admin用户），用户的请求才会被允许。

This permission is suitable if you want your API to only be accessible to a subset of trusted administrators.

如果您希望您的API只能被部分受信任的管理员访问，则此权限很适合你。

## IsAuthenticatedOrReadOnly

经过身份验证的权限或者仅有只读权限

The `IsAuthenticatedOrReadOnly` will allow authenticated users to perform any request.  Requests for unauthorised users will only be permitted if the request method is one of the "safe" methods; `GET`, `HEAD` or `OPTIONS`.

`IsAuthenticatedOrReadOnly`将允许经过身份验证的用户进行任何请求。 而未通过身份验证的用户只能使用“安全”方法（ `GET`, `HEAD` or `OPTIONS`）进行请求。

This permission is suitable if you want to your API to allow read permissions to anonymous users, and only allow write permissions to authenticated users.

如果您希望您的API允许匿名用户具有读取权限，只允许已通过身份验证的用户具有写入权限，那么你应该选择这种权限。

## DjangoModelPermissions

DjangoModel权限

This permission class ties into Django's standard `django.contrib.auth` [model permissions][contribauth].  This permission must only be applied to views that have a `.queryset` property set. Authorization will only be granted if the user *is authenticated* and has the *relevant model permissions* assigned.

该权限与Django 中标准的`django.contrib.auth`model 权限绑定。

该权限只能用于有`.queryset`属性的views。

只有用户通过了身份验证并且分配了相应的model权限，那么该用户才会拥有此权限。

* `POST` requests require the user to have the `add` permission on the model.
* `POST`请求需要用户对model具有`add`权限。
* `PUT` and `PATCH` requests require the user to have the `change` permission on the model.
* `PUT`和`PATCH`请求需要用户对model具有`change`权限。
* `DELETE` requests require the user to have the `delete` permission on the model.
* `DELETE`请求需要用户对model拥有`delete`权限。

The default behaviour can also be overridden to support custom model permissions.  For example, you might want to include a `view` model permission for `GET` requests.

如果你想自定义模型权限，你也可以重写默认行为。比如，你想为`GET`请求编写一个叫做`view`的model 权限。

To use custom model permissions, override `DjangoModelPermissions` and set the `.perms_map` property.  Refer to the source code for details.

如果你想自定义模型权限，请重写`DjangoModelPermissions`并设置`.perms_map`属性。 有关详细信息，请参阅源代码。

#### Using with views that do not include a `queryset` attribute.

使用没有`queryset`属性的views

If you're using this permission with a view that uses an overridden `get_queryset()` method there may not be a `queryset` attribute on the view. In this case we suggest also marking the view with a sentinel queryset, so that this class can determine the required permissions. 

如果你将此权限与重写了`get_queryset()`方法的view一起使用，那么该view可能会没有`queryset`属性。

如果是这种情况，我们建议为该view添加一个具有保护作用（能够保证该view仍然有`.queryset`属性，就像下面的例子一样）的queryset ，这样该view class 就能确定所需的权限了。



For example:

```Python
queryset = User.objects.none()  # Required for DjangoModelPermissions  
                                # DjangoModelPermission需要view具有该属性
```
## DjangoModelPermissionsOrAnonReadOnly

Similar to `DjangoModelPermissions`, but also allows unauthenticated users to have read-only access to the API.

和`DjangoModelPermission`相似，但是未通过身份验证的用户拥有只读权限。

## DjangoObjectPermissions

This permission class ties into Django's standard [object permissions framework][objectpermissions] that allows per-object permissions on models.  In order to use this permission class, you'll also need to add a permission backend that supports object-level permissions, such as [django-guardian][guardian].

该权限类与Django中的object permission framework绑定，允许models 的每个对象的所有权限。

为了使用此权限类，您还需要添加支持对象级权限的权限后端，例如django-guardian。

As with `DjangoModelPermissions`, this permission must only be applied to views that have a `.queryset` property or `.get_queryset()` method. Authorization will only be granted if the user *is authenticated* and has the *relevant per-object permissions* and *relevant model permissions* assigned.

和`DjangoModelPermission`一样，该权限也要求views具有`.queryset`属性或者`.get_queryset()`方法。只有在用户通过身份验证并且具有相关的每个对象权限和相关的模型权限后，那么该用户才会拥有该权限。

* `POST` requests require the user to have the `add` permission on the model instance.
* `POST`请求需要用户对model 实例拥有`add`权限。
* `PUT` and `PATCH` requests require the user to have the `change` permission on the model instance.
* `PUT`和`PATCH`请求需要用户对model 实例拥有`change`权限。
* `DELETE` requests require the user to have the `delete` permission on the model instance.
* `DELETE`请求需要用户对model 实例拥有`delete`权限。

Note that `DjangoObjectPermissions` **does not** require the `django-guardian` package, and should support other object-level backends equally well.

注意，`DjangoModelPermission`并不是只能添加`django-guardian`包，其他具有同等效果的对象级权限后端都可以添加。

As with `DjangoModelPermissions` you can use custom model permissions by overriding `DjangoObjectPermissions` and setting the `.perms_map` property.  Refer to the source code for details.

就像`DjangoModelPermission`一样，你可以通过重写`DjangoObjectPermission`并设置`.perms_map`属性来自定义model 权限。你可以通过查阅源码来了解更多细节。

---

**Note**: If you need object level `view` permissions for `GET`, `HEAD` and `OPTIONS` requests, you'll want to consider also adding the `DjangoObjectPermissionsFilter` class to ensure that list endpoints only return results including objects for which the user has appropriate view permissions.

如果您需要获取`GET`，`HEAD`和`OPTIONS`请求的对象级`view`权限，则还需要考虑添加`DjangoObjectPermissionsFilter`class，以确保接口只返回具有适当查看权限的用户的对象。

---

---

# Custom permissions

自定义权限

To implement a custom permission, override `BasePermission` and implement either, or both, of the following methods:

如果你想自定义权限，那么你需要重写`BasePermission`，你需要在你自定义的权限类里面实现下面方法中的一个或多个：

* `.has_permission(self, request, view)`
* `.has_object_permission(self, request, view, obj)`

The methods should return `True` if the request should be granted access, and `False` otherwise.

如果上面的方法返回`True`，那么请求将会获得权限；如果返回的是`False`，那请求将会因为权限不足而被拒绝。

If you need to test if a request is a read operation or a write operation, you should check the request method against the constant `SAFE_METHODS`, which is a tuple containing `'GET'`, `'OPTIONS'` and `'HEAD'`.  For example:

如果你想要知道一个请求是只读请求还是写入请求，那么你应该检查该请求方法是否在`SAFE_METHODS`中，`SAFE_METHODS`是一个包含`GET`、`HEAD`、`OPTIONS`的元组。

```python
if request.method in permissions.SAFE_METHODS:
    # Check permissions for read-only request
    # 如果是只读请求，将检查其是否具有只读权限
else:
    # Check permissions for write request
    # 如果是写入请求，将检查其是否具有写入权限
```

---

**Note**: The instance-level `has_object_permission` method will only be called if the view-level `has_permission` checks have already passed. Also note that in order for the instance-level checks to run, the view code should explicitly call `.check_object_permissions(request, obj)`. If you are using the generic views then this will be handled for you by default.

**注意**只有在视图级别`has_permission`检查通过时才会调用实例级别的`has_object_permission`方法。还要注意，为了运行实例级检查，我们要在视图代码中显式调用`.check_object_permissions(request，obj)`。如果你使用的是通用views，那么默认已经帮你进行了这些操作。

---

Custom permissions will raise a `PermissionDenied` exception if the test fails. To change the error message associated with the exception, implement a `message` attribute directly on your custom permission. Otherwise the `default_detail` attribute from `PermissionDenied` will be used.

如果请求没有相关权限，那么自定义权限类将会抛出`PermissionDenied`的异常。如果你想要自定义错误消息，那么你只需要在自定义权限类中设置消息属性即可。如果你没有自定义错误消息，那么默认将使用`PermissionDenied`的`default_detail`属性。

```python
from rest_framework import permissions
```

```python
class CustomerAccessPermission(permissions.BasePermission):
    message = 'Adding customers not allowed.'

    def has_permission(self, request, view):
         ...
```

## Examples

The following is an example of a permission class that checks the incoming request's IP address against a blacklist, and denies the request if the IP has been blacklisted.

下面这个例子中的权限类是用来检测IP黑名单的，如果请求的IP不在Blacklist里面，那么该请求可以正常进行请求操作，如果在Blacklist中，那么该请求将被拒绝。

```python
from rest_framework import permissions
```

```python
class BlacklistPermission(permissions.BasePermission):
    """
    Global permission check for blacklisted IPs.
    """

    def has_permission(self, request, view):
        ip_addr = request.META['REMOTE_ADDR']
        blacklisted = Blacklist.objects.filter(ip_addr=ip_addr).exists()
        return not blacklisted
```

As well as global permissions, that are run against all incoming requests, you can also create object-level permissions, that are only run against operations that affect a particular object instance.  For example:

除了针对所有传入请求运行的全局权限，您还可以创建对象级权限，这些权限仅针对影响特定对象实例的操作。

```python
class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Object-level permission to only allow owners of an object to edit it.
    Assumes the model instance has an `owner` attribute.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Instance must have an attribute named `owner`.
        return obj.owner == request.user
```

Note that the generic views will check the appropriate object level permissions, but if you're writing your own custom views, you'll need to make sure you check the object level permission checks yourself.  You can do so by calling `self.check_object_permissions(request, obj)` from the view once you have the object instance.  This call will raise an appropriate `APIException` if any object-level permission checks fail, and will otherwise simply return.

在通用的views里，我们对一部分对象级的权限进行了检查，所以如果你不使用通用views，那么你要记得在你的view里面进行适当的对象级权限检查。你可以在视图中调用`self.check_object_permissions(request，obj)`来完成此操作。如果请求未通过权限检查，那么将会抛出`APIException`异常，如果通过了权限检查，那么该请求继续执行。

Also note that the generic views will only check the object-level permissions for views that retrieve a single model instance.  If you require object-level filtering of list views, you'll need to filter the queryset separately.  See the [filtering documentation][filtering] for more details.

另请注意，通用views将仅检查检索单个模型实例的视图的对象级权限。如果你需要对list views进行对象级的过滤，那么你需要对queryset中的每个实例分别进行过滤。如果你想知道关于过滤更多信息，请参阅filtering documentation。

---

# Third party packages

第三方权限包

The following third party packages are also available.

在设置权限时，下面列出的第三方权限包也是值得你考虑的。

## Composed Permissions

The [Composed Permissions][composed-permissions] package provides a simple way to define complex and multi-depth (with logic operators) permission objects, using small and reusable components.

Composed Permissions提供了使用小的可重用组件来定义复杂和多深度（使用逻辑运算符）的权限对象的简单方法，

## REST Condition

The [REST Condition][rest-condition] package is another extension for building complex permissions in a simple and convenient way.  The extension allows you to combine permissions with logical operators.

REST Condition用于以简单方便的方式构建复杂权限的另一个扩展。 该扩展允许您将权限与逻辑运算符组合在一起。

## DRY Rest Permissions

The [DRY Rest Permissions][dry-rest-permissions] package provides the ability to define different permissions for individual default and custom actions. This package is made for apps with permissions that are derived from relationships defined in the app's data model. It also supports permission checks being returned to a client app through the API's serializer. Additionally it supports adding permissions to the default and custom list actions to restrict the data they retrieve per user.

DRY Rest Permission 为单个默认的操作以及自定义的操作提供了定义多种权限的功能。在你的APP中，你定义的数据模型之间的关系往往会派生出一些行为，而我们正好可以使用DRY Rest Permission为这些行为添加权限。它还支持通过API的序列化程序返回到客户端的权限检查。

## Django Rest Framework Roles

The [Django Rest Framework Roles][django-rest-framework-roles] package makes it easier to parameterize your API over multiple types of users.

Django Rest Framework Roles包使您可以更轻松地对你的API进行参数化，并且允许不同类型的user访问你的API。

## Django Rest Framework API Key

The [Django Rest Framework API Key][django-rest-framework-api-key] package allows you to ensure that every request made to the server requires an API key header. You can generate one from the django admin interface.

Django Rest Framework API Key包确保向服务器发出的每个请求都需要一个API key header。 您可以从django管理界面生成一个key。

[cite]: https://developer.apple.com/library/mac/#documentation/security/Conceptual/AuthenticationAndAuthorizationGuide/Authorization/Authorization.html
[authentication]: authentication.md
[throttling]: throttling.md
[filtering]: filtering.md
[contribauth]: https://docs.djangoproject.com/en/stable/topics/auth/customizing/#custom-permissions
[objectpermissions]: https://docs.djangoproject.com/en/stable/topics/auth/customizing/#handling-object-permissions
[guardian]: https://github.com/lukaszb/django-guardian
[get_objects_for_user]: http://pythonhosted.org/django-guardian/api/guardian.shortcuts.html#get-objects-for-user
[2.2-announcement]: ../topics/2.2-announcement.md
[filtering]: filtering.md
[drf-any-permissions]: https://github.com/kevin-brown/drf-any-permissions
[composed-permissions]: https://github.com/niwibe/djangorestframework-composed-permissions
[rest-condition]: https://github.com/caxap/rest_condition
[dry-rest-permissions]: https://github.com/Helioscene/dry-rest-permissions
[django-rest-framework-roles]: https://github.com/computer-lab/django-rest-framework-roles
[django-rest-framework-api-key]: https://github.com/manosim/django-rest-framework-api-key

source: throttling.py

# Throttling

> HTTP/1.1 420 Enhance Your Calm
>
> [Twitter API rate limiting response][cite]

Throttling is similar to [permissions], in that it determines if a request should be authorized.  Throttles indicate a temporary state, and are used to control the rate of requests that clients can make to an API.

As with permissions, multiple throttles may be used.  Your API might have a restrictive throttle for unauthenticated requests, and a less restrictive throttle for authenticated requests.

Another scenario where you might want to use multiple throttles would be if you need to impose different constraints on different parts of the API, due to some services being particularly resource-intensive.

Multiple throttles can also be used if you want to impose both burst throttling rates, and sustained throttling rates.  For example, you might want to limit a user to a maximum of 60 requests per minute, and 1000 requests per day.

Throttles do not necessarily only refer to rate-limiting requests.  For example a storage service might also need to throttle against bandwidth, and a paid data service might want to throttle against a certain number of a records being accessed.

Throttling(限制)与permissions(权限)很相似，它同样决定是否允许某一个请求通过。Throttles 用来临时控制客户端请求API的速率。

就像你使用permissions一样，你也可能会使用多个throttles。对于未通过身份认证的请求，API应该对这个请求有更多的限制，而对于通过身份认证的请求，限制则会较少（需要视实际情况而定）。

如果API中的某一部分服务特别占用资源时，你可能想要限制这些服务，此时你就需要针对API的不同部分配置不同的Throttles。

有时候你想要限制一个较小的时间间隔内的请求或者一个较长时间段内的请求，你此时需要多个Throttles。举个栗子：你想要限制用户访问服务器的速度，一分钟内的请求次数不能超过60次，一天最多请求1000次。

Throttles并不是只用在限制请求速度这方面，在其他的方面同样可以使用。比如，存储服务器可能需要限制带宽，付费服务器可能需要限制访问次数。



## How throttling is determined

As with permissions and authentication, throttling in REST framework is always defined as a list of classes.

Before running the main body of the view each throttle in the list is checked.
If any throttle check fails an `exceptions.Throttled` exception will be raised, and the main body of the view will not run.

与permissions和authentication一样，throttling也通常被定义为一系列类。

在执行view的主体代码之前，将先检查每一个throttle（`settings.py`中的`DEFAULT_THROTTLE_CLASSES`或者view中的`throttle_classes`）是否允许请求通过。

如果有一个throttle没有通过请求，则会抛出`exceptions.Throttled`异常，view的主体代码不会执行。



## Setting the throttling policy

throttling配置

The default throttling policy may be set globally, using the `DEFAULT_THROTTLE_CLASSES` and `DEFAULT_THROTTLE_RATES` settings.  For example.

默认，我们在`settings.py`中通过`DEFAULT_THOROTTLE_CLASSES`和`DEFAULT_THROTTLE_RATES`设置全局的throttles。

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```

The rate descriptions used in `DEFAULT_THROTTLE_RATES` may include `second`, `minute`, `hour` or `day` as the throttle period.

`DEFAULT_THROTTLE_RATES`中可以使用`second`、`minute`、`hour`、`day`来对限速度进行描述。

You can also set the throttling policy on a per-view or per-viewset basis,
using the `APIView` class-based views.

当然，你也可以针对某个view进行throttle设置。

```python
from rest_framework.response import Response
from rest_framework.throttling import UserRateThrottle
from rest_framework.views import APIView

class ExampleView(APIView):
    throttle_classes = (UserRateThrottle,)

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)
```

Or, if you're using the `@api_view` decorator with function based views.

```python
@api_view(['GET'])
@throttle_classes([UserRateThrottle])
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```



## How clients are identified

如何识别客户端

The `X-Forwarded-For` HTTP header and `REMOTE_ADDR` WSGI variable are used to uniquely identify client IP addresses for throttling.  If the `X-Forwarded-For` header is present then it will be used, otherwise the value of the `REMOTE_ADDR` variable from the WSGI environment will be used.

`X-Forwarded-For`HTTP header和`REMOTE_ADDR`WSGI变量作为被限制的客户端的唯一标识。

优先使用`X-Forwarder-For`HTTP header，如果没有该header，则使用`REMOTE_ADDR`WSGI变量。

If you need to strictly identify unique client IP addresses, you'll need to first configure the number of application proxies that the API runs behind by setting the `NUM_PROXIES` setting.  This setting should be an integer of zero or more.  If set to non-zero then the client IP will be identified as being the last IP address in the `X-Forwarded-For` header, once any application proxy IP addresses have first been excluded.  If set to zero, then the `REMOTE_ADDR` value will always be used as the identifying IP address.

如果你想要更加精确的知道客户端的ip地址，那么你首先要配置`NUM_PROXIES`（你的web应用使用的代理的数量），如果将它设置为非零，则HTTP header 中的`X-Forwarded-For`的第一个ip（排除了所有的代理IP后，只剩下客户端的IP）就是客户端的ip（但是有个问题，HTTP header中的信息是可以伪造的，所以有时候这个IP并不是真实的客户端IP）；如果将`NUM_PROXIES`设置为0，则将会使用`REMOTE_ADDR`的值作为客户端的IP（但这样如果客户端使用了代理，则该地址是代理服务器的IP，并不是客户端真实的ip）。

It is important to understand that if you configure the `NUM_PROXIES` setting, then all clients behind a unique [NAT'd](http://en.wikipedia.org/wiki/Network_address_translation) gateway will be treated as a single client.

如果你配置了`NUM_PROXIES`，那么理解相关的原理是很有必要的。如果多个客户端使用一个NAT与服务器通信，服务器根据IP将会把他们当做一个客户端来对待。

Further context on how the `X-Forwarded-For` header works, and identifying a remote client IP can be [found here][identifing-clients].

关于更多`X-Forwarder-For`的内容，你可以[found here][identifing-clients]。



## Setting up the cache

配置缓存

The throttle classes provided by REST framework use Django's cache backend.  You should make sure that you've set appropriate [cache settings][cache-setting].  The default value of `LocMemCache` backend should be okay for simple setups.  See Django's [cache documentation][cache-docs] for more details.

Throttle classes使用Django的缓存后台。所以你要确保你已经设置了恰当的缓存。如果你并不需要配置复杂的缓存，那么`LocMemCache`缓存后台的默认设置就已经足够满足你的需求了。如果你想配置更复杂的缓存，请参考Django的 [cache documentation][cache-docs] 。

If you need to use a cache other than `'default'`, you can do so by creating a custom throttle class and setting the `cache` attribute.  For example:

如果你不想使用缓存后台提供默认设置，你可以自定义throttle class，然后设置这个throttle class的`cache`属性。示例如下：

```python
class CustomAnonRateThrottle(AnonRateThrottle):
    cache = get_cache('alternate')
```

You'll need to remember to also set your custom throttle class in the `'DEFAULT_THROTTLE_CLASSES'` settings key, or using the `throttle_classes` view attribute.

当然，你需要记得在settings.py中设置`'DEFAULT_THROTTLE_CLASSES'`，或者在view中使用`throttle_classes`属性，来让你自定义的throttle class生效。

---

# API Reference

## AnonRateThrottle

The `AnonRateThrottle` will only ever throttle unauthenticated users.  The IP address of the incoming request is used to generate a unique key to throttle against.

The allowed request rate is determined from one of the following (in order of preference).

* The `rate` property on the class, which may be provided by overriding `AnonRateThrottle` and setting the property.
* The `DEFAULT_THROTTLE_RATES['anon']` setting.

`AnonRateThrottle` is suitable if you want to restrict the rate of requests from unknown sources.

`AnonRateThrottle`只对未通过设法认证的用户有限制作用。`AnonTateThrottle`将会根据请求的IP地址来生成一个与该IP唯一对应的key，然后对该IP进行限制。

允许的请求速率取决于下面的条件之一（下面的条件按照优先级从高到低排列）：

- 你在view中设置的`rate`属性，你也可以自定义Throttle class重写`AnonTateThrottle`，并在该Throttle class中设置`rate`属性。
- 你在settings.py中设置的`DEFAULT_THROTTLE_RATES['anon']`。

`ＡnonRateThrottle`非常适合对未知来源请求的请求速率的严格把控。



## UserRateThrottle

The `UserRateThrottle` will throttle users to a given rate of requests across the API.  The user id is used to generate a unique key to throttle against.  Unauthenticated requests will fall back to using the IP address of the incoming request to generate a unique key to throttle against.

The allowed request rate is determined from one of the following (in order of preference).

* The `rate` property on the class, which may be provided by overriding `UserRateThrottle` and setting the property.
* The `DEFAULT_THROTTLE_RATES['user']` setting.

An API may have multiple `UserRateThrottles` in place at the same time.  To do so, override `UserRateThrottle` and set a unique "scope" for each class.

`UserRateThrottle`将会在经过身份认证的用户的请求速率超过允许的请求速率时限制用户请求。user id 将被用来生成一个与该用户对应的唯一key，在需要限制的时候根据key对该用户进行限制。

For example, multiple user throttle rates could be implemented by using the following classes...

```python
class BurstRateThrottle(UserRateThrottle):
    scope = 'burst'

class SustainedRateThrottle(UserRateThrottle):
    scope = 'sustained'
```

...and the following settings.

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'example.throttles.BurstRateThrottle',
        'example.throttles.SustainedRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        'burst': '60/min',
        'sustained': '1000/day'
    }
}
```

`UserRateThrottle` is suitable if you want simple global rate restrictions per-user.

## ScopedRateThrottle

The `ScopedRateThrottle` class can be used to restrict access to specific parts of the API.  This throttle will only be applied if the view that is being accessed includes a `.throttle_scope` property.  The unique throttle key will then be formed by concatenating the "scope" of the request with the unique user id or IP address.

The allowed request rate is determined by the `DEFAULT_THROTTLE_RATES` setting using a key from the request "scope".

For example, given the following views...

```python
class ContactListView(APIView):
    throttle_scope = 'contacts'
    ...

class ContactDetailView(APIView):
    throttle_scope = 'contacts'
    ...

class UploadView(APIView):
    throttle_scope = 'uploads'
    ...
```

...and the following settings.

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.ScopedRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'contacts': '1000/day',
        'uploads': '20/day'
    }
}
```

User requests to either `ContactListView` or `ContactDetailView` would be restricted to a total of 1000 requests per-day.  User requests to `UploadView` would be restricted to 20 requests per day.

---

# Custom throttles

To create a custom throttle, override `BaseThrottle` and implement `.allow_request(self, request, view)`.  The method should return `True` if the request should be allowed, and `False` otherwise.

Optionally you may also override the `.wait()` method.  If implemented, `.wait()` should return a recommended number of seconds to wait before attempting the next request, or `None`.  The `.wait()` method will only be called if `.allow_request()` has previously returned `False`.

If the `.wait()` method is implemented and the request is throttled, then a `Retry-After` header will be included in the response.

## Example

The following is an example of a rate throttle, that will randomly throttle 1 in every 10 requests.

```python
import random
```

```python
class RandomRateThrottle(throttling.BaseThrottle):
    def allow_request(self, request, view):
        return random.randint(1, 10) != 1
```

[cite]: https://dev.twitter.com/docs/error-codes-responses
[permissions]: permissions.md
[identifing-clients]: http://oxpedia.org/wiki/index.php?title=AppSuite:Grizzly#Multiple_Proxies_in_front_of_the_cluster
[cache-setting]: https://docs.djangoproject.com/en/stable/ref/settings/#caches
[cache-docs]: https://docs.djangoproject.com/en/stable/topics/cache/#setting-up-the-cache

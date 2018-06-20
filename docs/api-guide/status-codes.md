source: status.py

# Status Codes

> 418 I'm a teapot - Any attempt to brew coffee with a teapot should result in the error code "418 I'm a teapot".  The resulting entity body MAY be short and stout.
>
> 418 I'm a teapot - 当你想要使用茶壶泡咖啡的时候就会返回错误码"418 I'm a teapot"。
>
> 结果可能太短并且不够简洁。
>
> &mdash; [RFC 2324][rfc2324], Hyper Text Coffee Pot Control Protocol

Using bare status codes in your responses isn't recommended.  REST framework includes a set of named constants that you can use to make your code more obvious and readable.

我们不建议你在使用纯粹的数字做状态码，别人想要搞懂它代表什么意思需要费些力气。

REST framework提供了一组能够清晰的表达状态的状态码，使用他们能让你的代码易读。

```python
from rest_framework import status
from rest_framework.response import Response

def empty_view(self):
    content = {'please move along': 'nothing to see here'}
    return Response(content, status=status.HTTP_404_NOT_FOUND)
```

The full set of HTTP status codes included in the `status` module is listed below.

所有的HTTP状态码都在`status`模块中。

The module also includes a set of helper functions for testing if a status code is in a given range.

`status`模块也包含一些用于测试的功能函数。

```python
from rest_framework import status
from rest_framework.test import APITestCase

class ExampleTestCase(APITestCase):
    def test_url_root(self):
        url = reverse('index')
        response = self.client.get(url)
        self.assertTrue(status.is_success(response.status_code))
```


For more information on proper usage of HTTP status codes see [RFC 2616][rfc2616]
and [RFC 6585][rfc6585].

## Informational - 1xx

This class of status code indicates a provisional response.  There are no 1xx status codes used in REST framework by default.

这类状态码表示临时响应。REST framework默认不使用1XX状态码。

    HTTP_100_CONTINUE
    HTTP_101_SWITCHING_PROTOCOLS

## Successful - 2xx

This class of status code indicates that the client's request was successfully received, understood, and accepted.

这类状态码表示成功接收到了客户端的请求，理解该请求的意思，并且同意该请求。

    HTTP_200_OK
    HTTP_201_CREATED
    HTTP_202_ACCEPTED
    HTTP_203_NON_AUTHORITATIVE_INFORMATION
    HTTP_204_NO_CONTENT
    HTTP_205_RESET_CONTENT
    HTTP_206_PARTIAL_CONTENT
    HTTP_207_MULTI_STATUS

## Redirection - 3xx

This class of status code indicates that further action needs to be taken by the user agent in order to fulfill the request.

这类状态码表示用户需要进一步进行操作来完成请求。

    HTTP_300_MULTIPLE_CHOICES
    HTTP_301_MOVED_PERMANENTLY
    HTTP_302_FOUND
    HTTP_303_SEE_OTHER
    HTTP_304_NOT_MODIFIED
    HTTP_305_USE_PROXY
    HTTP_306_RESERVED
    HTTP_307_TEMPORARY_REDIRECT

## Client Error - 4xx

The 4xx class of status code is intended for cases in which the client seems to have erred.  Except when responding to a HEAD request, the server SHOULD include an entity containing an explanation of the error situation, and whether it is a temporary or permanent condition.

4xx类的状态码表示客户端出现了错误。除了HEAD 以外，响应其他的请求都应该包含有关错误的信息，以及标明该错误是临时性的还是永久性的。

    HTTP_400_BAD_REQUEST
    HTTP_401_UNAUTHORIZED
    HTTP_402_PAYMENT_REQUIRED
    HTTP_403_FORBIDDEN
    HTTP_404_NOT_FOUND
    HTTP_405_METHOD_NOT_ALLOWED
    HTTP_406_NOT_ACCEPTABLE
    HTTP_407_PROXY_AUTHENTICATION_REQUIRED
    HTTP_408_REQUEST_TIMEOUT
    HTTP_409_CONFLICT
    HTTP_410_GONE
    HTTP_411_LENGTH_REQUIRED
    HTTP_412_PRECONDITION_FAILED
    HTTP_413_REQUEST_ENTITY_TOO_LARGE
    HTTP_414_REQUEST_URI_TOO_LONG
    HTTP_415_UNSUPPORTED_MEDIA_TYPE
    HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE
    HTTP_417_EXPECTATION_FAILED
    HTTP_422_UNPROCESSABLE_ENTITY
    HTTP_423_LOCKED
    HTTP_424_FAILED_DEPENDENCY
    HTTP_428_PRECONDITION_REQUIRED
    HTTP_429_TOO_MANY_REQUESTS
    HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE
    HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS

## Server Error - 5xx

Response status codes beginning with the digit "5" indicate cases in which the server is aware that it has erred or is incapable of performing the request.  Except when responding to a HEAD request, the server SHOULD include an entity containing an explanation of the error situation, and whether it is a temporary or permanent condition.

5xx类的状态码表示服务器端发生错误。除了HEAD 以外，响应其他的请求都应该包含有关错误的信息，以及标明该错误是临时性的还是永久性的。

    HTTP_500_INTERNAL_SERVER_ERROR
    HTTP_501_NOT_IMPLEMENTED
    HTTP_502_BAD_GATEWAY
    HTTP_503_SERVICE_UNAVAILABLE
    HTTP_504_GATEWAY_TIMEOUT
    HTTP_505_HTTP_VERSION_NOT_SUPPORTED
    HTTP_507_INSUFFICIENT_STORAGE
    HTTP_511_NETWORK_AUTHENTICATION_REQUIRED

## Helper functions

The following helper functions are available for identifying the category of the response code.

这些帮助函数可以用来判断响应状态码的类别。

    is_informational()  # 1xx
    is_success()        # 2xx
    is_redirect()       # 3xx
    is_client_error()   # 4xx
    is_server_error()   # 5xx

[rfc2324]: http://www.ietf.org/rfc/rfc2324.txt
[rfc2616]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
[rfc6585]: http://tools.ietf.org/html/rfc6585

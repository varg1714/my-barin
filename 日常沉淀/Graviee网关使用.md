# Graviee 网关使用

## 1. 限流过滤

对于请求的限流，如果想要使用令牌桶的限流算法，可以使用 `Rate-Limit` 限流器，对于漏桶算法可以使用 `Spike-Arrest` 限流器。`Quota` 限流器可以实现每月/天类似的调用次数限制。

## 2. 请求修改

可以基于 Groovy 脚本实现请求的预处理，如编辑请求的 header、修改请求的参数之类。

以下示例说明：

- 根据 cookie 中的值来编辑请求信息
    ```groovy
    // 获取当前的Cookie头部值
    def cookiesHeader = request.headers.get('Cookie')
    if (cookiesHeader) {
        // 分割Cookie字符串为单独的Cookies，提前join一次兼容请求中有多个cookie头的情况
        def cookies = cookiesHeader.join(';').split(';')
          
        // 移除特定名字的cookie
        cookies = cookies.findAll { cookie ->
            cookie != null && cookie !='' && !cookie.trim().startsWith('minerva_tkn=')
        }
        
        // 将剩余的Cookies重新拼接成字符串
        def newCookiesHeader = cookies.join(';')
        
        // 设置新的Cookie头部值
        request.headers.'Cookie' = newCookiesHeader
    }
    // 添加大会员的特殊标记
    request.headers.'authToken' = 'lm'
    request.headers.remove('minerva_tkn')
    request.headers.remove('feign-request')
    ```
- 往上游携带 requestId 信息
    ```groovy
    // 获取当前的Cookie头部值
    def requestId = request.headers.get('X-LF-RequestId')
    if (!requestId) {
        // 设置新的Cookie头部值
        request.headers.'X-LF-RequestId' = request.headers.get('X-Gaia-Request-Id')
    }
    ```

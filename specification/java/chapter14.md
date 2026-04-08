# （十四）WEB规约

**1.【强制】使用名词代替动词。**

资源操作与HTTP方法映射关系：

-   获取资源：GET

-   创建资源：POST

-   全量更新资源：PUT

-   部分更新资源：PATCH

-   删除资源：DELETE

-   检查资源是否存在：HEAD

正例：GET /cars/1

反例：GET /getCars/1

例外：当某个操作（如下载和上传）无对应HTTP方法时，在最后加上对应的动词，如 download / uplaod，幂等操作使用 GET ，非幂等操作使用 POST 。

正例：GET /users/1/downlaod

正例：POST /users/upload

反例：POST /users/1/download

反例：DELETE /users/1/inactivate

**2.【强制】GET 操作不允许修改资源状态。**

正例：POST /users/1/activate

反例：GET /users/1/activate

**3.【强制】资源名称使用复数。**

正例：POST /users

正例：GET /groups/1/users

反例：POST /user

反例：GET /group/1/users

**4.【强制】使用HTTP头以声明接受的及返回的数据格式**

Content-Type: application/json

使用框架中的`@GetJsonMapping`注解可自动添加上例中 json 的 Content-Type，`@PostJsonMapping`、`@PutJsonMapping`则分别用以接收 POST 及 PUT 请求体为 json 的请求。

**5.【强制】分页、排序及过滤**

**5.1. 分页参数为`page`和`size`，分别表示第几页及返回多少条记录。**

**5.2. 用户输入`size`小于某个最低值（比如 10 ），则前端返回最低值给后端；后端发现用户输入的`size`大于某个阀值（如 500 ），则返回该阀值条数记录给用户。**

**5.3. 使用sort参数对多个字段进行排序。**

    GET /cars?sort=-manufactorer,+model

`+`表示升序，`-`表示降序，不写则表示升序，多个字段以`,`分割。

**5.4. 分页返回结构参照如下格式。**

    {
        "total" : 100,
        "data" : [
            {
                "id": 1,
                "name": "John"
            },
            {
                "id": 2,
                "name": "Peter"
            },
            {
                "id": 3,
                "name": "Marry"
            }
        ]
    }

**5.5. 使用fields字段表示仅希望返回fields中提供的字段。**

    GET /cars?fields=manufacturer,model,id,color

该用法一般只推荐在后端内部服务之间调用时使用，前端调用后端接口不推荐使用这种写法，以避免数据库字段信息暴露及攻击风险。

**6.【强制】对API进行版本管理。**

对外暴露的接口加上 api 的前缀。

    /api/v1/users/1

在 HTTP 头中而不是请求路径中加入版本号，有利于向前兼容。可参考Github API设计。

**7.【强制】返回正确的HTTP状态码。**

在 HTTP 规范中，2xx 状态码表示请求成功返回，3xx 表示重定向，4xx 表示客户端错误，5xx 表示服务端错误。

特别注意，不允许后端服务在捕获异常后再返回 2xx 状态码。

反例：

    try {
        service.findById(1);
        return ResponseEntity.ok();
    } catch (Throable t) {
        return ResponseEntity.ok();
    }

<table>
<caption>RESTful常见状态码及使用场景说明</caption>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">HTTP状态码</th>
<th style="text-align: left;">说明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><p>200</p></td>
<td style="text-align: left;"><p>正常返回，一般在GET、非创建资源POST或批量POST创建资源时使用。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>201</p></td>
<td style="text-align: left;"><p>资源创建成功，在POST创建资源请求时使用。</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>202</p></td>
<td style="text-align: left;"><p>请求已被接受，一般对于耗时处理的请求时使用，后端往往会用另外一个线程对请求进行处理，以免长时间阻塞请求。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>204</p></td>
<td style="text-align: left;"><p>请求成功，但无响应内容，一般在更新及删除成功时使用。</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>304</p></td>
<td style="text-align: left;"><p>客户端可以使用已缓存的数据。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>400</p></td>
<td style="text-align: left;"><p>一般表示请求语义有误或请求参数有误。除非进行修改，否则客户端不应该重复提交这个请求。</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>401</p></td>
<td style="text-align: left;"><p>当前请求需要用户进行认证。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>403</p></td>
<td style="text-align: left;"><p>用户认证通过，但无权限访问当前请求的资源。</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>404</p></td>
<td style="text-align: left;"><p>访问的资源不存在。此处不仅仅指访问的URL不存在，也有可能请求的资源在数据库中不存在，此时也应返回此状态码。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>405</p></td>
<td style="text-align: left;"><p>请求行中指定的请求方法不能被用于请求相应的资源。也有可能是后端错误标注了HTTP方法所致。</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>409</p></td>
<td style="text-align: left;"><p>创建的资源已存在、更新的资源已被其他用户更新或更新及删除的资源已被其他用户删除。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>422</p></td>
<td style="text-align: left;"><p>用户及参数等校验都已通过，但是请求不满足业务约束，如所创建账号不能超过100等。</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>500</p></td>
<td style="text-align: left;"><p>所有服务端非预期的异常都应该返回该状态码。</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>503</p></td>
<td style="text-align: left;"><p>所依赖的外部服务若无法提供服务或返回异常，则应该返回该异常。</p></td>
</tr>
</tbody>
</table>

RESTful常见状态码及使用场景说明

**8.【强制】服务端发生错误时，返回给前端的响应信息必须包含 HTTP 状态码、code、message三个部分。**

    {
        "code": "10000",
        "message": "Unknown error"
    }

可使用框架中的`RestErrorResponse`对象，对于未捕获异常框架会自动处理并返回相应结构体。

**9.【强制】在前后端交互的 JSON 格式数据中，所有的 key 必须为小写字母开始的 lowerCamelCase 风格，符合英文表达习惯，且表意完整**

正例：errorCode / errorMessage / assetStatus / menuList / orderList / configFlag

反例：ERRORCODE / ERROR\_CODE / error\_message / error-message / errormessage / ErrorMessage / msg

**10.【强制】对于需要使用超大整数的场景，服务端一律使用`String`字符串类型返回，禁止使用`Long`类型。**

Java 服务端如果直接返回`Long`整型数据给前端，Javascript 会自动转换为`Number`类型(注:此类型为双精度浮点数，表示原理与取值范围等同于 Java 中的`Double`)。`Long`类型能表示的最大值是 263-1，在取值范围之内，超过 253 (9007199254740992)的数值转化为 Javascript 的`Number`时，有些数值会产生精度损失。

在`Long`取值范围内，任何 2 的指数次的整数都是绝对不会存在精度损失的，所以说精度损失是一个概率问题。若浮点数尾数位与指数位空间不限，则可以精确表示任何整数，但很不幸，双精度浮点数的尾数位只有 52 位。

反例：通常在订单号或交易号大于等于 16 位，大概率会出现前后端单据不一致的情况，比如`orderId`: 362909601374617692，前端拿到的值却是: 362909601374617660。

**11.【强制】HTTP请求通过URL传递参数时，不能超过2048字节。**

不同浏览器对于 URL 的最大长度限制略有不同，并且对超出最大长度的处理逻辑也有差异，2048 字节是取所有浏览器的最小值。

反例：某业务将退货的商品 id 列表放在 URL 中作为参数传递，当一次退货商品数量过多时，URL 参数超长，传递到后端的参数被截断，导致部分商品未能正确退货。

**12.【强制】HTTP 请求通过 body 传递内容时，必须控制长度，超出最大长度后，后端解析会出错。**

nginx 默认限制是 1MB，tomcat 默认限制为 2MB，当确实有业务需要传较大内容时，可以通过调大服务器端的限制。

**13.【强制】服务器内部重定向必须使用 forward；外部重定向地址必须使用 URL 统一代理模块生成，否则会因线上采用 HTTPS 协议而导致浏览器提示“不安全”，并且还会带来 URL 维护不一致的问题。**

**14.【强制】前后端的日期时间格式统一使用ISO格式，时区统一为UTC时区。**

正例：2011-12-03T10:15:30Z

反例：2011-12-03T10:15:30+01:00

反例：2011-12-03 10:15:30

**15.【推荐】在URL中体现资源的关系。**

正例：GET /zoos/1/animals

例外：避免在URL中表达三层及以上的层级的资源关系，这种情况推荐使用参数过滤。

正例：GET /users?country=china&province=shanghai&city=shanghai

反例：GET /countries/china/provinces/shanghai/cities/shanghai/users

**16.【推荐】使用HATEOAS。**

RESTful 的终极形态，在资源的表达中包含了链接信息。客户端可以根据链接来发现可以执行的动作。

正例：

    {
        "id" : 1,
        "body" : "My first blog post",
        "postdate" : "2015-05-30T21:41:12.650Z",
        "_links" : {
            "self": { "href": "http://blog.example.com/posts/1" },
            "comments": { "href": "http://blog.example.com/posts/1/comments", "totalcount" : 20 },
            "tags": { "href": "http://blog.example.com/posts/1/tags" }
        }
    }

对于POST创建资源请求，后端应该返回Location头以供前端直接查询所创建资源的详情。

正例：

    Location: http://localhost:8080/hermesapi/v1/accounts/1

Spring提供了对HATEOAS的支持，感兴趣的同学可以了解下。

**17.【推荐】服务器返回信息必须被标记是否可以缓存，如果缓存则客户端可能会重用之前的请求结果。**

可参考Spring中`WebRequest`的`checkNotModified`等方法。

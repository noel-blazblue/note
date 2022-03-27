参考文章：
- https://zh.javascript.info/fetch-crossorigin#yong-yu-jian-dan-qing-qiu-de-cors


浏览器会自动进行 CORS 通信，实现CORS通信的关键是后端。只要后端实现了 CORS，就实现了跨域。


**多年来，来自一个网站的脚本无法访问另一个网站的内容。**

这个简单有力的规则是互联网安全的基础。例如，来自 `[hacker.com](http://hacker.com/)` 的脚本无法访问 `[gmail.com](http://gmail.com/)` 上的用户邮箱。基于这样的规则，人们感到很安全。

在那时候，JavaScript 并没有任何特殊的执行网络请求的方法。它只是一种用来装饰网页的玩具语言而已。

但是 Web 开发人员需要更多功能。人们发明了各种各样的技巧去突破该限制，并向其他网站发出请求。

## 之前浏览器的一些跨域方法
### 使用表单
```html
<!-- 表单目标 -->
<iframe name="iframe"></iframe>

<!-- 表单可以由 JavaScript 动态生成并提交 -->
<form target="iframe" method="POST" action="http://another.com/…">
  ...
</form>
```

提交到 `iframe` 是为了停留在当前页面。
表单可以将数据发送到任何地方，GET/POST 请求都行。但是由于禁止从其他网站访问 `<iframe>` 中的内容，因此就无法读取响应。

### 使用 script
`script` 可以具有任何域的 `src`，例如 `<script src="[http://another.com/…](http://another.com/%E2%80%A6)">`。也可以执行来自任何网站的 `script`。

```js
// 1. 声明处理天气数据的函数
function gotWeather({ temperature, humidity }) {
  alert(`temperature: ${temperature}, humidity: ${humidity}`);
}

// 然后我们创建一个特性（attribute）为 `src="[http://another.com/weather.json?callback=gotWeather](http://another.com/weather.json?callback=gotWeather)"` 的 `<script>` 标签，使用我们的函数名作为它的 `callback` URL-参数。

let script = document.createElement('script');
script.src = `http://another.com/weather.json?callback=gotWeather`;
document.body.append(script);

// 我们期望来自服务器的回答看起来像这样：
gotWeather({
  temperature: 25,
  humidity: 78
});

```

4.   当远程脚本加载并执行时，`gotWeather` 函数将运行，并且因为它是我们的函数，我们就有了需要的数据。


不久之后，网络方法出现在了浏览器 JavaScript 中。

起初，跨源请求是被禁止的。但是，经过长时间的讨论，跨源请求被允许了，但是任何新功能都需要服务器明确允许，以特殊的 header 表述。

之前，没有人能够设想网页能发出这样的请求。因此，可能仍然存在有些 Web 服务将非标准方法视为一个信号：“这不是浏览器”。它们可以在检查访问权限时将其考虑在内。

因此，为了避免误解，任何“非标准”请求 —— 浏览器不会立即发出在过去无法完成的这类请求。即在它发送这类请求前，会先发送“预检（preflight）”请求来请求许可。

## 跨域请求
有两种类型的跨源请求：

1.  简单的请求。
2.  所有其他请求。

1.  [简单的方法](http://www.w3.org/TR/cors/#simple-method)：GET，POST 或 HEAD
2.  [简单的 header](http://www.w3.org/TR/cors/#simple-header) —— 仅允许自定义下列 header：
    -   `Accept`，
    -   `Accept-Language`，
    -   `Content-Language`，
    -   `Content-Type` 的值为 `application/x-www-form-urlencoded`，`multipart/form-data` 或 `text/plain`。

任何其他请求都被认为是“非简单请求”。例如，具有 `PUT` 方法或 `API-Key` HTTP-header 的请求就不是简单请求。

**本质区别在于，可以使用 `<form>` 或 `<script>` 进行“简单请求”，而无需任何其他特殊方法。**

### 简单请求
#### request header
如果一个请求是跨源的，浏览器始终会向其添加 `Origin` header。
例如，如果我们从 `https://javascript.info/page` 请求 `https://anywhere.com/request`，请求的 header 将会如下：
```http
GET /request
Host: anywhere.com
Origin: https://javascript.info
```

浏览器在这里扮演受被信任的中间人的角色：
1.  它确保发送的跨源请求带有正确的 `Origin`。
2.  它检查响应中的许可 `Access-Control-Allow-Origin`，如果存在，则允许 JavaScript 访问响应，否则将失败并报错。

![[Pasted image 20210604123518.png]]

#### response header
对于跨源请求，默认情况下，JavaScript 只能访问“简单” response header：
-   `Cache-Control`
-   `Content-Language`
-   `Content-Type`
-   `Expires`
-   `Last-Modified`
-   `Pragma`

要授予 JavaScript 对任何其他 response header 的访问权限，服务器必须发送 `Access-Control-Expose-Headers` header。它包含一个以逗号分隔的应该被设置为可访问的非简单 header 名称列表。

```http
200 OK
Content-Type:text/html; charset=UTF-8
Content-Length: 12345
API-Key: 2c9de507f2c54aa1
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Expose-Headers: Content-Length,API-Key
```

### 非简单请求
我们可以使用任何 HTTP 方法：不仅仅是 `GET/POST`，也可以是 `PATCH`，`DELETE` 及其他。

之前，没有人能够设想网页能发出这样的请求。因此，可能仍然存在有些 Web 服务将非标准方法视为一个信号：“这不是浏览器”。它们可以在检查访问权限时将其考虑在内。

因此，为了避免误解，任何“非标准”请求 —— 浏览器不会立即发出在过去无法完成的这类请求。即在它发送这类请求前，会先发送“预检（preflight）”请求来请求许可。

预检请求使用 `OPTIONS` 方法，它没有 body，但是有两个 header：

-   `Access-Control-Request-Method` header 带有非简单请求的方法。
-   `Access-Control-Request-Headers` header 提供一个以逗号分隔的非简单 HTTP-header 列表。

如果服务器同意处理请求，那么它会进行响应，此响应的状态码应该为 200，没有 body，具有 header：

-   `Access-Control-Allow-Origin` 必须为 `*` 或进行请求的源（例如 `[https://javascript.info](https://javascript.info/)`）才能允许此请求。
-   `Access-Control-Allow-Methods` 必须具有允许的方法。
-   `Access-Control-Allow-Headers` 必须具有一个允许的 header 列表。
-   另外，header `Access-Control-Max-Age` 可以指定缓存此权限的秒数。因此，浏览器不是必须为满足给定权限的后续请求发送预检。

![[Pasted image 20210604124058.png]]

对于一个跨源的 `PATCH` 请求（此方法经常被用于更新数据）：
```js
let response = await fetch('https://site.com/service.json', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
    'API-Key': 'secret'
  }
});
```

这里有三个理由解释为什么它不是一个简单请求（其实一个就够了）：
-   方法 `PATCH`
-   `Content-Type` 不是这三个中之一：`application/x-www-form-urlencoded`，`multipart/form-data`，`text/plain`。
-   “非简单” `API-Key` header。

#### 预检请求
在发送我们的请求前，浏览器会自己发送如下所示的预检请求：
```http
OPTIONS /service.json
Host: site.com
Origin: https://javascript.info
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Content-Type,API-Key
```

-   方法：`OPTIONS`。
-   路径 —— 与主请求完全相同：`/service.json`。
-   特殊跨源头：
    -   `Origin` —— 来源。
    -   `Access-Control-Request-Method` —— 请求方法。
    -   `Access-Control-Request-Headers` —— 以逗号分隔的“非简单” header 列表。

预检请求发生在“幕后”，它对 JavaScript 不可见。
JavaScript 仅获取对主请求的响应，如果没有服务器许可，则获得一个 error。

#### 预检响应
```http
200 OK
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Allow-Methods: PUT,PATCH,DELETE
Access-Control-Allow-Headers: API-Key,Content-Type,If-Modified-Since,Cache-Control
Access-Control-Max-Age: 86400
```

现在，浏览器可以看到 `PATCH` 在 `Access-Control-Allow-Methods` 中，`Content-Type,API-Key` 在列表 `Access-Control-Allow-Headers` 中，因此它将发送主请求。

如果 `Access-Control-Max-Age` 带有一个表示秒的数字，则在给定的时间内，预检权限会被缓存。上面的响应将被缓存 86400 秒，也就是一天。在此时间范围内，后续请求将不会触发预检。假设它们符合缓存的配额，则将直接发送它们。

#### 实际请求
预检成功后，浏览器现在发出主请求。这里的算法与简单请求的算法相同。

主请求具有 `Origin` header（因为它是跨源的）：
```http
PATCH /service.json
Host: site.com
Content-Type: application/json
API-Key: secret
Origin: https://javascript.info
```

#### 实际响应
服务器不应该忘记在主响应中添加 `Access-Control-Allow-Origin`。成功的预检并不能免除此要求：
```http
Access-Control-Allow-Origin: https://javascript.info
```


### credentials
默认情况下，由 JavaScript 代码发起的跨源请求不会带来任何凭据（cookies 或者 HTTP 认证（HTTP authentication））。
例如，`fetch('[http://another.com](http://another.com/)')` 不会发送任何 cookie
要在 `fetch` 中发送凭据，我们需要添加 `credentials: "include"` 选项，像这样：
```js
fetch('http://another.com', {
  credentials: "include"
});
```
现在，`fetch` 将把源自 `[another.com](http://another.com/)` 的 cookie 和我们的请求发送到该网站。

如果服务器同意接受 **带有凭据** 的请求，则除了 `Access-Control-Allow-Origin` 外，服务器还应该在响应中添加 header `Access-Control-Allow-Credentials: true`。
```http
200 OK
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Allow-Credentials: true
```


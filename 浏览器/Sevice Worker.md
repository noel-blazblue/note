参考文章：https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API

Service workers 本质上充当 Web 应用程序、浏览器与网络（可用时）之间的代理服务器。这个 API 旨在创建有效的离线体验，它会拦截网络请求并根据网络是否可用采取来适当的动作、更新来自服务器的的资源。它还提供入口以推送通知和访问后台同步 API。

Service worker是一个注册在指定源和路径下的事件驱动[worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker)。它采用JavaScript控制关联的页面或者网站，拦截并修改访问和资源请求，细粒度地缓存资源。你可以完全控制应用在特定情形（最常见的情形是网络不可用）下的表现。

Service worker运行在worker上下文，因此它不能访问DOM。相对于驱动应用的主JavaScript线程，它运行在其他线程中，所以不会造成阻塞。它设计为完全异步，同步API（如[XHR](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)和[localStorage](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/Web_Storage_API)）不能在service worker中使用。

出于安全考量，Service workers只能由HTTPS承载，毕竟修改网络请求的能力暴露给中间人攻击会非常危险。在Firefox浏览器的[用户隐私模式](https://support.mozilla.org/zh-CN/kb/%E9%9A%90%E7%A7%81%E6%B5%8F%E8%A7%88)，Service Worker不可用。

## 使用场景
Service workers也可以用来做这些事情：

-   后台数据同步
-   响应来自其它源的资源请求
-   集中接收计算成本高的数据更新，比如地理位置和陀螺仪信息，这样多个页面就可以利用同一组数据
-   在客户端进行CoffeeScript，LESS，CJS/AMD等模块编译和依赖管理（用于开发目的）
-   后台服务钩子
-   自定义模板用于特定URL模式
-   性能增强，比如预取用户可能需要的资源，比如相册中的后面数张图片

未来service workers能够用来做更多使web平台接近原生应用的事。 值得关注的是，其他标准也能并且将会使用service worker，例如:

-   [后台同步](https://github.com/slightlyoff/BackgroundSync)：启动一个service worker即使没有用户访问特定站点，也可以更新缓存
-   [响应推送](https://developer.mozilla.org/zh-CN/docs/Web/API/Push_API)：启动一个service worker向用户发送一条信息通知新的内容可用
-   对时间或日期作出响应
-   进入地理围栏
# 背景

cookie 是前端网络知识中非常重要的一个概念，现在做一个系统性的总结，其中包括 cookie 的概念以及和安全结合理解 cookie 的应用。

# cookie 概念

cookie 的英文意思有两个一个是饼干，另一个是坚强的人。网络层面携带一部分信息这个概念命名为 cookie 的原因可以看这个
https://www.inlife.co.uk/why-are-cookies-called-cookies/

cookie 是存储在浏览器中由服务器产生的数据信息(部分 cookie 也可能是由网页 javascript 代码注入)，常用来存储用户信息等。

cookie 的存储是通过一个字符串来完成的


```javascript
"name=xxx;age=123"

```

既然 cookie 这个概念代表一部分数据，那么下面我们将讨论数据的来源、存储以及应用

# cookie 的来源

- 服务器: 这个存储数据的机制一个很好的优点就是服务器可以通过 cookie 直接在浏览器中存储相关数据
- javascript 可以新增、修改(部分 cookie)

# cookie 的存储

cookie 的存储以及修改会涉及到 cookie 的配置问题，下面对相关配置进行总结

## Expires

cookie 的最长有效时间，如果没有设置这个属性，浏览器关闭时 cookie 会被清除

## Max-Age

经过多少秒 cookie 过期，如果也存在 Expires 属性，已 Max-Age 为准。也就是说 Max-Age 比 Expires 权重要高。

## Domain

域名规定 cookie 可被发送的域名,如果不设置，默认为当前页面的 host。如果设置了，子域名都是允许的。

比如 Domain = a.com，那么域名 b.a.com 就是允许访问这个 cookie 的。

## Path

页面路径和域名的设置原理基本一致，/ 字符被认为是文件名的分隔符。 如果你设定 Path=/docs  那么

- /docs
- /docs/web
都是可以访问 cookie 的
- 


## Secure

设定之后，只有是 http 协议才能访问这个 cookie

## HttpOnly

设置后 javascript 无法访问这个 cookie

## SameSite

- Strict 严格模式，禁止所有跨域的 cookie 发送
- None 完全允许，允许所有跨域的 cookie 发送
- Lax 宽松模式，允许部分跨域 cookie 发送

## cookie 的跨域使用

当 cookie 存储到浏览器当中后，剩下我们要说的就是 cookie 的使用，

在浏览器存储由接口返回的 cookie 后，此后的每一个请求，只要是满足 cookie 的设置，cookie 都会被 http 请求携带。

cookie 的使用中的难点是跨域请求中，对于 cookie 的使用方式，我在这一部分会重点对跨域请求的 cookie 携带做一个总结。

## cors 接口跨域请求

对于cookie 的使用，一个主动地使用场景就是请求跨域的接口，如果这个跨域的接口需要携带 cookie 有以下两个方面需要考虑

### withCredentials

**跨域请求接口，浏览器默认是不会携带 cookie 的**

在调用接口后，我们需要指定一个属性 withCredentials

```javascript
const invocation = new XMLHttpRequest();
const url = 'https://bar.other/resources/credentialed-content/';

function callOtherDomain() {
  if (invocation) {
    invocation.open('GET', url, true);
    // 在指定这个属性后，接口调用才有可能携带 cookie
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send();
  }
}


```

cors 跨域请求分两种一种是简单请求,一种是复杂请求 (不理解请自行baidu)

- 如果是简单请求,当设置此属性后,跨域请求会直接携带相关的 cookie
- 复杂请求会等待 option 预请求回来后，才会携带相关 cookie


### SameSite

SameSite 属性是浏览器除了 withCredentials 判断跨域请求是否可以携带此 cookie 的另一个机制。

**浏览器的跨域请求必须同时满足这两个机制，才能将 cookie 发出**。

除了我们在 javascript 中使用 XMLHttpRequest 以及 fetch。我们还会在 html 中引入一下标签，常用的比如 img、iframe 等。其中这些资源也会涉及到 cookie 发送的问题，这些问题往往被我们所忽视：

|请求类型|示例|strict|None|lax|
|--|---|---|---|---|
|链接|< a href=""></a>|否|是|是|
|预加载|<link ref="prerender" href="" />|否|是|是|
|get 表单|<form method="get">|否|是|是|
|post 表单|<form method="post">|否|是|否|
|iframe|<iframe src="">|否|是|否|
|AJAX|get("")|否|是|否|
|image|<img src="">|否|是|否|

其中需要特别注意的就是 a 标签，当点击标签后，如果 same-site 属性为 strict，跳转过去的页面是不会携带 cookie 的。所以会造成很多网页的登录状态失效，这个要特别注意，所以我觉得默认的 lax 其实是比较合适的，在安全性以及易用性上面做出了很好的平衡。

# cookie 与安全

在学习使用 egg 的过程中，会涉及到一些关于 cookie 安全的问题。下面对这些 cookie的安全问题做一个总结

## csrf-token

csrf (cross-site request forgery) 跨站请求伪造。一句话解释就是，攻击者会利用 cookie 信息，进行请求伪造。在第三方网页上伪造真实请求，进行网络攻击。

特点：

- 攻击一般发生在第三方网站，而不是被攻击的网站。
- 攻击者利用受害者 cookie 信息，冒充受害者身份，进行网络交互。

基于以上两个特点，我们就要特别关注跨域 cookie 的存储以及使用问题。

### 解决方式

第一种就是 same-site 属性。当然只有支持这个属性的浏览器才可以。如果是版本较老的浏览器就不行了。

第二种方式就是 csrf-token

csrf-token 简单说就是在 cookie 中存储一个新的字段，通过网络请求页面时服务器注入一个加密的字符串，在之后的网络请求时，都会携带这个字符串，如果服务器判断字符串不合法，不进行后端操作以及返回数据。

#### csrf-token 的实现也有两种方式。

1. 第一种就是 session 存储, 服务器会在内存或者数据库存储 session 数据。
2. 第二种方式是规定所有接口都要携带一个新的 http header 携带 csrf-token 数值，并且浏览器也会自动在 cookie 字段中携带 csrf-token 数值。服务器会在请求时比对这两个 token 是否一致来判断是否来自于第三方页面。

我比较喜欢第二种方式，因为可以减轻服务器的压力，实现也比较简单。

### cookie 防篡改

cookie 存储在浏览器中，javascript、用户手动都是可以进行修改的。

为了方式 cookie 被篡改，可以在服务器下发 cookie 的时候，同时下发一个根据内容加密的 cookie 字段。
比如

```

set-cookie name=008; path=/; httponly
set-cookie name.sig=BKz_FtEld6gVjNwSzdNZAXZCq3n2Vf7VcHISiEBp7oc; path=/; httponly

```

其中 name.sig 就是防止篡改的字段。当服务器接收到 name cookie时会同时和 sig 进行比对，如果不一致，就意味着 cookie 被篡改了


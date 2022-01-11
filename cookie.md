## 背景

cookie 是前端网络知识中非常重要的一个概念，现在做一个系统性的总结，其中包括 cookie 的概念以及和安全结合理解 cookie 的应用。

## cookie 概念

cookie 的英文意思有两个一个是饼干，另一个是坚强的人。网络层面携带一部分信息这个概念命名为 cookie 的原因可以看这个
https://www.inlife.co.uk/why-are-cookies-called-cookies/

cookie 是存储在浏览器中由服务器产生的数据信息(部分 cookie 也可能是由网页 javascript 代码注入)，常用来存储用户信息等。

cookie 的存储是通过一个字符串来完成的

```javascript
"name=xxx;age=123"

```

既然 cookie 这个概念代表一部分数据，那么下面我们将讨论数据的来源、存储以及应用

## cookie 的来源

- 服务器: 这个存储数据的机制一个很好的优点就是服务器可以通过 cookie 直接在浏览器中存储相关数据
- javascript 可以新增、修改(部分 cookie)

## cookie 的存储

cookie 的存储以及修改会涉及到 cookie 的配置问题，下面对相关配置进行总结

### Expires

cookie 的最长有效时间，如果没有设置这个属性，浏览器关闭时 cookie 会被清除

### Max-Age

经过多少秒 cookie 过期，如果也存在 Expires 属性，已 Max-Age 为准。也就是说 Max-Age 比 Expires 权重要高。

### Domain

### Path

### Secure

### HttpOnly

### SameSite




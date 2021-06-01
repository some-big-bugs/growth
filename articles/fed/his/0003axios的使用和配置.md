<h1> 全栈的自我修养: Axios 的简单使用 </h1>

> You should never judge something you don't understand.<br>
> 你不应该去评判你不了解的事物。<br>

**Table of Contents**

- [介绍](#介绍)
- [简单使用](#简单使用)
  - [GET](#get)
  - [DELETE](#delete)
  - [PUT](#put)
  - [POST](#post)
  - [PATCH](#patch)
  - [汇总](#汇总)
- [使用 `application/x-www-form-urlencoded`](#使用-applicationx-www-form-urlencoded)
  - [方式一：使用 `URLSearchParams`](#方式一使用-urlsearchparams)
  - [方式二：使用 `qs` 进行编码](#方式二使用-qs-进行编码)
- [使用 `multipart/form-data`](#使用-multipartform-data)
- [Response 结构](#response-结构)
- [Config 常用配置](#config-常用配置)
- [参考](#参考)


# 介绍

Axios 是一个基于 Promise 的 HTTP 库，可以用在浏览器和 node.js 中。

Github开源地址： https://github.com/axios/axios

如果你原来用过 `jQuery` 应该还记的 `$.ajax` 方法吧

# 简单使用

如果按照`HTTP`方法的语义来暴露资源，那么接口将会拥有安全性和幂等性的特性，例如GET和HEAD请求都是安全的， 无论请求多少次，都不会改变服务器状态。而GET、HEAD、PUT和DELETE请求都是幂等的，无论对资源操作多少次， 结果总是一样的，后面的请求并不会产生比第一次更多的影响。

下面列出了 `GET`，`DELETE`，`PUT`, `PATCH` 和 `POST` 的典型用法:

## GET

> `axios#get(url[, config])`<br>
> 从方法声明可以看出<br>
> 1. 第一个参数`url`必填，为请求的url
> 2. 第二个参数 `config` 选填, 关于`config` 的属性见下文


`GET` 方法用来查询服务资源, 不应该在这里对服务资源进行修改


1. 使用get 方法进行请求，参数可以直接拼接在 url 中

```js 
axios.get('/user?id=12345')
  .then(response => {
    // 如果成功返回（http 状态码在 200~300），则可获取对应的 response
    console.log(response);
  })
  .catch(error => {
    // 异常
    console.log(error);
  })
  .then(() => {
    // always executed
  });
```

1. 使用get 方法进行请求，参数单独作为一个对象传入, 该参数会拼接在url 中

```js

let request_params = { id: 123456 }

axios.get('/user', {
    params: request_params
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

## DELETE

> `axios#delete(url[, config])`<br>
> 从方法声明可以看出
> 1. 第一个参数`url`必填，为请求的url
> 2. 第二个参数 `config` 选填, 关于`config` 的属性见下文

`DELETE` 用来删除一个资源，不安全但幂等

1. 使用 DELETE 方法进行请求，参数可以直接拼接在 url 中

```js 
axios.delete('/user?id=12345')
  .then(response => {
    // 如果成功返回（http 状态码在 200~300），则可获取对应的 response
    console.log(response);
  })
  .catch(error => {
    // 异常
    console.log(error);
  })
  .then(() => {
    // always executed
  });
```

1. 使用 DELETE 方法进行请求，参数单独作为一个对象传入, 该参数会拼接在url 中

```js

let request_params = { id: 123456 }

axios.delete('/user', {
    params: request_params
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

3. 使用 DELETE 方法进行请求，参数单独作为一个对象传入, 该参数会在请求体中

```js

let request_params = { id: 123456 }

axios.delete('/user', {
    data: request_params
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

## PUT

> `axios#put(url[, data[, config]])`<br>
> 从方法声明可以看出
> 1. 第一个参数`url`必填，为请求的url
> 2. 第二个参数`data`选填，为请求的参数，且在请求体中
> 2. 第二个参数 `config` 选填, 关于`config` 的属性见下文

1. 不安全但幂等
2. 通过替换的方式更新资源

> 常见使用方式

1. 使用 PUT 方法进行请求，参数可以直接拼接在 url 中

**更新资源**

```js 
axios.put('/user?id=12345&name=abc')
  .then(response => {
    // 如果成功返回（http 状态码在 200~300），则可获取对应的 response
    console.log(response);
  })
  .catch(error => {
    // 异常
    console.log(error);
  })
  .then(() => {
    // always executed
  });
```

1. 使用 PUT 方法进行请求，参数单独作为一个对象传入, 该参数会在请求体中

```js

let request_params = { id: 123456, name: "abc" }

axios.put('/user', request_params,
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

## POST

> `axios#post(url[, data[, config]])`<br>
> 从方法声明可以看出
> 1. 第一个参数`url`必填，为请求的url
> 2. 第二个参数`data`选填，为请求的参数，且在请求体中
> 2. 第二个参数 `config` 选填, 关于`config` 的属性见下文

1. 不安全且不幂等
2. 创建资源

> 常见使用方式

1. 使用 POST 方法进行请求，参数可以直接拼接在 url 中

**创建id为123456的用户**

```js 
axios.post('/user?id=12345&name=abc')
  .then(response => {
    // 如果成功返回（http 状态码在 200~300），则可获取对应的 response
    console.log(response);
  })
  .catch(error => {
    // 异常
    console.log(error);
  })
  .then(() => {
    // always executed
  });
```

1. 使用 POST 方法进行请求，参数单独作为一个对象传入, 该参数会在请求体中

```js

let request_params = { id: 123456, name: "abc" }

axios.post('/user', request_params,
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

## PATCH

> `axios.patch(url[, data[, config]])`<br>
> 从方法声明可以看出
> 1. 第一个参数`url`必填，为请求的url
> 2. 第二个参数`data`选填，为请求的参数，且在请求体中
> 2. 第二个参数 `config` 选填, 关于`config` 的属性见下文

1. 不安全且不幂等
2. 在服务器更新资源（客户端提供改变的属性，部分更新）

> 常见使用方式

1. 使用 PATCH 方法进行请求，参数可以直接拼接在 url 中

**更新id为123456的用户资源**

```js 
axios.patch('/user?id=12345&name=abc')
  .then(response => {
    // 如果成功返回（http 状态码在 200~300），则可获取对应的 response
    console.log(response);
  })
  .catch(error => {
    // 异常
    console.log(error);
  })
  .then(() => {
    // always executed
  });
```

1. 使用 PATCH 方法进行请求，参数单独作为一个对象传入, 该参数会在请求体中

```js

let request_params = { id: 123456, name: "abc" }

axios.patch('/user', request_params,
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

## 汇总

从上面的示例中可以看出

```js
axios.get(url[, config])
axios.delete(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```

其中 `POST`、`PUT`、`PATCH` 的使用方式是一致的，只是`方式名`和 `http method` 存在差异, 那他们的区别在什么地方呢

```
GET：从服务器取出资源（一项或多项）。
POST：在服务器新建一个资源。
PUT：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH：在服务器更新资源（客户端提供改变的属性）。
DELETE：从服务器删除资源。
```


# 使用 `application/x-www-form-urlencoded`

在默认情况下，data 中数据采用了 JSON 序列化方式，即 `Content-Type: application/json`, 如果想使用 `application/x-www-form-urlencoded`, 则需要做特殊处理

## 方式一：使用 `URLSearchParams`

```js
const params = new URLSearchParams();
params.append('id', '123456');
params.append('name', 'abc');
axios.post('/user', params);
```

其中 URLSearchParams 存在兼容问题，具体可见[caniuse](https://www.caniuse.com/#feat=urlsearchparams)

## 方式二：使用 `qs` 进行编码

```js
import qs from 'qs';
axios.post('/user', qs.stringify({ id: 123456, name: "abc" }));
```

# 使用 `multipart/form-data`

```js
const form = new FormData();
form.append('id', 123456);
form.append('name', "abc");

axios.post('user', form, { headers: form.getHeaders() })
```

# Response 结构

```js
{
  // `data` is the response that was provided by the server
  // response 返回数据
  data: {},

  // `status` is the HTTP status code from the server response
  // 状态码
  status: 200,

  // `statusText` is the HTTP status message from the server response
  // 状态码对应的标准message
  statusText: 'OK',

  // `headers` the HTTP headers that the server responded with
  // All header names are lower cased and can be accessed using the bracket notation.
  // Example: `response.headers['content-type']`
  // 响应头
  headers: {},

  // `config` is the config that was provided to `axios` for the request
  config: {},

  // `request` is the request that generated this response
  // It is the last ClientRequest instance in node.js (in redirects)
  // and an XMLHttpRequest instance in the browser
  request: {}
}
```

# Config 常用配置

```js
{
  // `url` is the server URL that will be used for the request
  url: '/user',

  // `method` is the request method to be used when making the request
  method: 'get', // default

  // `baseURL` will be prepended to `url` unless `url` is absolute.
  // It can be convenient to set `baseURL` for an instance of axios to pass relative URLs
  // to methods of that instance.
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` allows changes to the request data before it is sent to the server
  // This is only applicable for request methods 'PUT', 'POST', 'PATCH' and 'DELETE'
  // The last function in the array must return a string or an instance of Buffer, ArrayBuffer,
  // FormData or Stream
  // You may modify the headers object.
  transformRequest: [function (data, headers) {
    // Do whatever you want to transform the data

    return data;
  }],

  // `transformResponse` allows changes to the response data to be made before
  // it is passed to then/catch
  transformResponse: [function (data) {
    // Do whatever you want to transform the data

    return data;
  }],

  // `headers` are custom headers to be sent
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` are the URL parameters to be sent with the request
  // Must be a plain object or a URLSearchParams object
  params: {
    ID: 12345
  },

  // `paramsSerializer` is an optional function in charge of serializing `params`
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function (params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` is the data to be sent as the request body
  // Only applicable for request methods 'PUT', 'POST', 'DELETE , and 'PATCH'
  // When no `transformRequest` is set, must be of one of the following types:
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - Browser only: FormData, File, Blob
  // - Node only: Stream, Buffer
  data: {
    firstName: 'Fred'
  },
  
  // syntax alternative to send data into the body
  // method post
  // only the value is sent, not the key
  data: 'Country=Brasil&City=Belo Horizonte',

  // `timeout` specifies the number of milliseconds before the request times out.
  // If the request takes longer than `timeout`, the request will be aborted.
  timeout: 1000, // default is `0` (no timeout)

  // `withCredentials` indicates whether or not cross-site Access-Control requests
  // should be made using credentials
  withCredentials: false, // default

  // `auth` indicates that HTTP Basic auth should be used, and supplies credentials.
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` indicates the type of data that the server will respond with
  // options are: 'arraybuffer', 'document', 'json', 'text', 'stream'
  //   browser only: 'blob'
  responseType: 'json', // default

  // `responseEncoding` indicates encoding to use for decoding responses (Node.js only)
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // default


  // `maxContentLength` defines the max size of the http response content in bytes allowed in node.js
  maxContentLength: 2000,

  // `maxBodyLength` (Node only option) defines the max size of the http request content in bytes allowed
  maxBodyLength: 2000,

  // `validateStatus` defines whether to resolve or reject the promise for a given
  // HTTP response status code. If `validateStatus` returns `true` (or is set to `null`
  // or `undefined`), the promise will be resolved; otherwise, the promise will be
  // rejected.
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },
}
```

更多配置参考 `https://github.com/axios/axios`

# 参考

1. https://github.com/axios/axios
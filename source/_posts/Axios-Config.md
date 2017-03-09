---
layout: _posts
title: 'Axios的配置'
date: 2017-01-29 20:25:08
tags: [Axios]
thumbnail: http://okkula0y9.bkt.clouddn.com/20160130.jpg
---

> 以前写Vue项目的时候都是使用vue-resource做为项目ajax库，在11月份的某一天尤大微博的更新表示ajax的库应该是通用的，放弃了对vue-resource的技术支持，推荐使用[axios][1]。

# Axios的配置

![此处输入图片的描述][2]
既然尤大推荐的应该有过人之处，好吧于是在新的项目上开始使用Axios,开启这段学习（踩坑）的历程。


  [1]: https://github.com/mzabriskie/axios
  [2]: http://upload-images.jianshu.io/upload_images/1987062-b3255d564903d3d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
  
## 安装
``` javascript
 npm install axios
```

##  使用
Axios和其他的ajax库都是很类似的，提供了2种使用的方式一种是直接使用实例方法的如：
下面是实例的所有可用方法，方法中的config会与axios实例中的config合并。（实例可以将一些通用的config先配置好）
``` javascript
//axios.get
//axios.post

axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

##   Config 配置
Axios的配置参数很多，我们来一一了解

- url —— 用来向服务器发送请求的url
- method —— 请求方法，默认是GET方法
-  baseURL —— 基础URL路径，假如url不是绝对路径，如https://some-domain.com/api/v1/login?name=jack,那么向服务器发送请求的URL将会是baseURL + url。
-  transformRequest —— transformRequest方法允许在请求发送到服务器之前修改该请求，此方法只适用于PUT、POST和PATCH方法中。而且，此方法最后必须返回一个string、ArrayBuffer或者Stream。
-  transformResponse —— transformResponse方法允许在数据传递到then/catch之前修改response数据。此方法最后也要返回数据。
- headers —— 发送自定义Headers头文件，头文件中包含了http请求的各种信息。
-  params —— params是发送请求的查询参数对象，对象中的数据会被拼接成url?param1=value1&param2=value2。
paramsSerializer —— params参数序列化器。
- data —— data是在发送POST、PUT或者PATCH请求的数据对象。
- timeout —— 请求超时设置，单位为毫秒
- withCredentials —— 表明是否有跨域请求需要用到证书
- adapter —— adapter允许用户处理更易于测试的请求。返回一个Promise和一个有效的response
- auth —— auth表明提供凭证用于完成http的身份验证。这将会在headers中设置一个Authorization授权信息。自定义Authorization授权要设置在headers中。
- responseType —— 表示服务器将返回响应的数据类型，有arraybuffer、blob、document、json、text、stream这6个类型，默认是json类似数据。
- xsrfCookieName —— 用作 xsrf token 值的 cookie 名称
- xsrfHeaderName —— 带有 xsrf token 值 http head 名称
- onUploadProgress —— 允许在上传过程中的做一些操作
- onDownloadProgress —— 允许在下载过程中的做一些操作
- maxContentLength —— 定义了接收到的response响应数据的最大长度。
- validateStatus —— validateStatus定义了根据HTTP响应状态码决定是否接收或拒绝获取到的promise。如果 validateStatus 返回 true (或设置为 null 或 undefined ),promise将被接收;否则,promise将被拒绝。
- maxRedirects —— maxRedirects定义了在node.js中redirect的最大值，如果设置为0，则没有redirect。
- httpAgent —— 定义在使用http请求时的代理
- httpsAgent —— 定义在使用https请求时的代理
- proxy —— proxy定义代理服务器的主机名和端口，auth
- cancelToken —— cancelToken定义一个 cancel token 用于取消请求


##  Response 返回
当我们ajax获取数据成功后会返回一个response对象，它包含了以下内容
``` javascript

{
  // 服务器返回的数据
  data: {},
  // HTTP状态吗
  status: 200,
  // 服务器返回的消息
  statusText: 'OK',
  // 返回头
  headers: {},
  // 在返回我们的配置
  config: {}
}
```

##  统一Config配置
在接口测试中，我们经常需要切换线上环境和测试环境，这里我们都可以通过Config来配置，这样我们所有的发起的请求都是通过这个基本的URL走了。

``` javascript
axios.defaults.timeout = 5000;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8';
axios.defaults.baseURL = 'http://www.xxxx.xxx/api';
// axios.defaults.baseURL = 'http://192.168.1.129:8383';
```


##  Interceptors 拦截器
这里我必须重点介绍，在我们发起大量的请求时候，需要对请求做统一的处理那就用到它了。笔者在使用了`vue-resource`和`axios`之后亲身比较，`axios`的配置更加人性化。
官方的API上这样介绍
> You can intercept requests or responses before they are handled by then or catch.
您可以拦截请求或响应之前，他们处理的操作或者异常


### request统一处理操作
如果是POST的请求，配置中可不能用`params`字段了，需要使用`data`字段。
这里有个小地方需要注意，POST的传参需要序列化，不然服务端不会正确的接收哦，会报错。所以这里我们要对request的数据进行一次序列化。这里我用了`qs`,大家需要install一下

``` javascript
import axios from 'axios'
import qs from 'qs'

axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8';

//POST传参序列化
axios.interceptors.request.use((config) => {
  if(config.method  === 'post'){
    config.data = qs.stringify(config.data);
  }
  return config;
},(error) =>{
   alert("错误的传参");
  return Promise.reject(error);
});
```


### response统一处理操作
也就是说我们可以统一的在发起请求前，或者获得数据，对其进行统一的操作。这点非常的高效，在笔者的项目中，接口会返回一个code,就和微信API一样，code为200代表返回请求数据正确为其它时就自动跳出弹窗打印消息即可。

``` javascript
//code状态码200判断
axios.interceptors.response.use((res) =>{
  if(res.data.code != '200'){
    alert(res.data.msg);
    return Promise.reject(res);
  }
  return res;
}, (error) => {
  alert("网络异常");
  return Promise.reject(error);
});
```

如果发生这些错误了我要结束当前的Promise所以返回一个`Promise.reject(res)`，停止Promise队列下面的操作,如果有对于Promise不熟悉的童鞋请自行搜索`Promise`(这里还遇到了一个小坑最后会介绍)


## 我的配置
好了介绍了这么多介绍下我的axios配置的文件设置吧,这个文件名是`config/http.js`
``` javascript

import axios from 'axios'
import qs from 'qs'
import * as _ from './whole'

axios.defaults.timeout = 5000;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8';
axios.defaults.baseURL = 'http://www.guinaben.com:8070';
// axios.defaults.baseURL = 'http://192.168.1.129:8383';

//POST传参序列化
axios.interceptors.request.use((config) => {
  if(config.method  === 'post'){
    config.data = qs.stringify(config.data);
  }
  return config;
},(error) =>{
   _.toast("错误的传参");
  return Promise.reject(error);
});

//code状态码200判断
axios.interceptors.response.use((res) =>{
  if(res.data.code != '200'){
    _.toast(res.data.msg);
    return Promise.reject(res);
  }
  return res;
}, (error) => {
  _.toast("网络异常");
  return Promise.reject(error);
});

export default axios;
```


发起的请求
``` javascript
import axios from 'config/http'

axios({
  method:'get',
  url: 'xxxx/xxxxx',
  params: {
    "textbook_id":id,
    "token":token
  }
})
.then((response) => {
  resolve(response);
})


axios({
  method:'post',
  url: 'teacher/pwd/resetByMobile',
  data: {
   "textbook_id":id,
    "token":token
  }
})
.then((response) => {
    resolve(response);
})
  
  
```

## 一定要看
因为这里我使用的`Promise`,所以在安卓4.4.3一下的手机还是不支持Promise的，所以会报错。需要引入` npm install babel-polyfill`和` npm install babel-runtime`，在入口文件上加上即可。
``` javascript
import 'babel-polyfill' 
```

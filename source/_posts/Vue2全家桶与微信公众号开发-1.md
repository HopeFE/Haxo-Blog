layout: _posts
title: ''
date: 2017-03-06 16:51:30
tags: [微信开发,Vue]
---

> 此项目本身有一个APP了，为了方便将APP和微信端数据打通，需要用户微信和APP用户绑定。在开发的过程中单页面的模式在微信JS API的配置踩了很多坑，特别是IOS。由于本人表述能力和篇幅有限Orz，这里只介绍关键的实现步骤和代码，有些安全的地方和路由地方处理当时比较暴力没有细化，还望交流指导。


## 阅读需知
- 默认您已经了解了微信授权登陆、微信JS API配置及微信支付的整体流程[微信帮助文档][1]
- Vue2.0（[Vue][2]、[VueRouter][3]、[Vuex][4]）全家桶基本知识，能熟悉70%以上的API
- [Axios][5]（Http请求组件）的基础知识（若不熟悉的可以看笔者的这2篇文章[axios全攻略][6],[Vuex2和Axios的配合][7]）
本来准备写一篇的，后来在写优化的时候发现东西比较多，还是分为2篇去写吧。第一篇主要写关于权限和微信配置方面的关键点。第二篇写关于性能和代码优化上的关键点。

## 技术选型
- 移动前端Vue组件库[Vux][8]
- [Vue][9]、[VueRouter][10]、[Vuex][11]Vue三件套
- 路由和Vuex同步组件[Vuex-router-sync][12]

**VueRouter选用的是Hash模式，避免每次都需要去注册WxConfig**

## 注意
1、有些页面比如下单页面是不能分享的，在JS API内要配置该页面分享的是一个可以通过访问的页面如系统首页
2、该系统主要是在微信中应用的，在授权登录时，授权登陆页面会判断是否是微信内核（不得不说这节省了我一大笔开发，没有权限的时候直接往这跳就好了。简单暴力）
3、每个关键点都能引申很多的知识，这里篇幅有限，不在详细介绍

# 实现过程
## 技术点
1. 用户授权登陆，服务端从授权页面获取到OpenId（加密处理过的）后打开一个特定的URL（前端提供功能保存OPENID，回跳）,此页面从URL上取到URL在保存在Store中。
2. 分享出去的页面最好都配授权页面登陆页面，在授权登陆页面的URL上可以配置自己需要回跳的页面，方便服务端有针对性的二次跳转。
3. 接口返回状态码若为401，则表示没有权限，在Axios内跳转到登陆页面
4. 若检测到系统内没有OpenId则跳转到授权登陆页面（防止有人把URL复制出去分享在微信中打开报错）

## 登陆逻辑流程图
![此处输入图片的描述][13]

# 关键技术点

## 授权登陆网址
``` html
https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxf565864b6a1a358d&
redirect_uri=http%3a%2f%2fpeifei.xxxx.com%2fnoa%2ftoken%3fpage%3dhttp%3a%2f%2fpeifei.xxx.com/index&response_type=code&scope=snsapi_userinfo&state=123#wechat_redirect
```
网址需要实现的功能：
- 获得OPENID传给前端(后端实现)
- 授权登陆后能跳转到想去的页面（后端跳转后前端路由控制）

## 从URL上获取OPENID
**vue-router和vuex的共同作用**

``` javascript
import store from '../../store'     //从根目录中引入store

export default [
  {
    path: '/', 
    component:R_INDEX,
    children: [
      {
        path: 'index/',
        redirect:'/'
      }
    ]
  },
  {
    //服务端一律跳转到这个URL上
    path: '/home/:id/:redirectUrl/', redirect: to => {
      /**
      * 通过dispatch触发保存openid的action
      * 将URL上的OPENID保存到store中
      */
      store.dispatch({
        type: 'setOpenId',
        amount: to.params.id
      })
      //在回跳到需要来访的正确页面
      return `/${to.params.redirectUrl}/`
    }
  }
]
```

## 接口的处理
**axios、vuex的使用结合**
此部分需要`Promise`和`Axios`的知识，若不熟悉请参阅笔者这2篇文章
1. [axios全攻略][14]
2. [Vuex2和Axios的配合][15]

### 基本设置
``` javascript
/* 
*  1.超时处理
*  2.post设置
*  3.开发环境与正式环境的区别
*/
axios.defaults.timeout = 5000
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8'
axios.defaults.baseURL =  (process.env.NODE_ENV == 'development' ? 'http://192.168.1.15:8080/' : 'http://www.xxxx.com/')
```
### 全局请求处理

``` javascript
/* 
*  1.请求拦截，全局增加token
*  2.post设置
*/
axios.interceptors.request.use((config) => {
  if(config.method  === 'post' || config.method  === 'put' ){
    //这里使用了qs这个库去序列化数据
    config.data = qs.parse(config.data,{arrayFormat:'brackets'})
  }
  //全局追加openid
  config.params = (
      Object.assign((config.params ? config.params : {}),{"SESSION":store.state.common.openid})
  )
  return config
},(error) =>{
  Vue.$vux.toast.show({text: '非法输入',type:'text',time:1000})
  return Promise.reject(error)
})
```

### 全局接收处理
``` javascript
/* 
*  1.正确请求接口后，接口内返回的code若不为20000（前后端接口的规定），则表示是错误的参数
*  2.若接口报错，并且报错的http状态码为401（前后端接口的规定）则表示用户没有该接口的权限，
*  跳转到登陆页面
*/
axios.interceptors.response.use((res) =>{
  if(res.data.code != '20000'){
    Vue.$vux.toast.show({text:res.data.message,type:'text',time:1000})
    return Promise.reject(res)
  }
  return res;
}, (error) => {
  if(error.response){
    switch (error.response.status){
      case 401:
        window.location.href = `http://${window.document.location.host}/?#/login/`
        break
      default:
        Vue.$vux.toast.show({text:'网络异常',type:'text',time:1000})
    }
  }
  return Promise.reject(error)
})
```
## 页面合法性检查
因为整套环境都是由前端去控制页面路由，这里有很多地方需要我们去做权限的验证完善程序的健壮性，这对前端的考验很大。
虽然分享出去的页面处理比较暴力都是授权的链接，还是担心有人会复制URL出去在微信中打开所以需要做以下处理。

1. 验证是否在微信端
2. store中是否存在openid，是否授权登陆后的进来的
3. 无权限操作的页面应该能返回正确的地方

### 检测是否授权登陆
我将授权登陆后的openid存在了store中，所以每次进行**路由跳转**的时候我只要检测store中是否存在openid若不存在则直接跳转到授权登录页面，授权登陆后服务端判断此openid是否存在，若存在则跳转到来访页面，不存在则跳转到login页面。

``` javascript
router.beforeEach((to, from, next) => {
  if(store.state.common.openId){
    next();
  }else{
    window.location.href="https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxf565864b6a1a358d&redirect_uri=http%3a%2f%2fpeifei.qmant.com%2fnoa%2ftoken%3fpage%3dhttp%3a%2f%2fpeifei.qmant.com/index&response_type=code&scope=snsapi_userinfo&state=123#wechat_redirect"
  }
})
```

### 是否在微信端
授权登陆页面会检测，若不在微信端会提示如下。
![此处输入图片的描述][16]

### 无权限页面
后端openid进行检测，若是无效的或者不对的openid会在请求接口的时候返回401，接口接收到401后，会跳转到登陆注册页面。

## 微信支付的坑
> 在遇到这个问题时困扰了好久，然后这篇博文[开发单页应用(SPA)时候遇到的微信支付授权目录的坑][17]给了我指导，在此感谢作者

### 路由模式
前端技术选型用的vuejs+vue-router，vue-router使用hashbang模式（使用hashbang也是为了避免微信jssdk的wx.config签名的坑）。在调用微信支付的时候(IOS)遇到提示"URL未注册"，这通常是因为没有在微信支付后台正确配置授权目录的问题，但是我遇到并不是这个。
我在调试的时候发现唤起微信支付时，IOS内打印日志中的URL和实际中的URL不一样安卓却是好的，我不知道是不是微信的BUG。后来在网上搜寻答案，发现是下面这个问题：

### 问题原因
首先把当前页面叫做`Current Page`。当我们从微信别的地方点击链接呼出微信浏览器时所落在的页面、或者点击微信浏览器的刷新按钮时所刷新的页面，我们叫做`Landing Page`。
举个例子，我们从任何地方点击链接进入页面A，然后依次浏览到B、C、D，那么Current Page就是D，而Landing Page是A，如果此时我们在D页面点击一下浏览器的刷新按钮，那么Landing Page就变成了D（以上均是在单页应用的环境下，即以hashbang模式通过js更改浏览器路径，直接href跳转的不算）。

问题来了，在iOS和安卓下呼出微信支付的时候，微信支付判断当前路径的规则分别是：

IOS：Landing Page
安卓：Current Page

这就意味着，在ios环境下，任何一个页面都有可能成为支付页面（因为我无法预知和控制用户在哪个页面点微信浏览器的刷新按钮，或是用户通过哪个连接从外部进入到系统）。

### 解决
``` html
3个页面用到微信支付：
http://example.com/#/cart/index
http://example.com/#/order/orderlist
http://example.com/#/order/orderinfo
```
上述的3个链接根本不行啊，因为微信授权目录必须配置到最后一级目录，配置在根目录不行。

将所有的路由#前加了一个？，于是微信浏览器妥妥的把井号“#”后面的内容给去掉了
``` html
原来路由链接：
http://example.com/#/cart/index
现在路由链接：
http://example.com/?#/cart/index
```
我们只要将授权目录设置到根目录`http://example.com/`即可


  [1]: https://mp.weixin.qq.com/wiki/17/c0f37d5704f0b64713d5d2c37b468d75.html
  [2]: http://cn.vuejs.org/guide/
  [3]: http://vuex.vuejs.org/zh-cn/index.html
  [4]: http://vuex.vuejs.org/zh-cn/index.html
  [5]: https://github.com/mzabriskie/axios
  [6]: https://ygxdxx.coding.me/2017/02/27/axios%E5%85%A8%E6%94%BB%E7%95%A5/
  [7]: https://ygxdxx.coding.me/2017/02/01/Vuex2%E4%B8%8EAxios%E5%BC%80%E5%8F%91/
  [8]: https://vux.li/#/
  [9]: http://cn.vuejs.org/guide/
  [10]: http://vuex.vuejs.org/zh-cn/index.html
  [11]: http://vuex.vuejs.org/zh-cn/index.html
  [12]: https://github.com/vuejs/vuex-router-sync
  [13]: http://okkula0y9.bkt.clouddn.com/wvue17_3_5.jpg
  [14]: https://ygxdxx.coding.me/2017/02/27/axios%E5%85%A8%E6%94%BB%E7%95%A5/
  [15]: https://ygxdxx.coding.me/2017/02/01/Vuex2%E4%B8%8EAxios%E5%BC%80%E5%8F%91/
  [16]: http://okkula0y9.bkt.clouddn.com/wx_2017_3_6.jpg
  [17]: http://blog.csdn.net/liufeng520/article/details/51354741


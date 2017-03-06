---
layout: _posts
title: ''
date: 2017-02-01 16:42:40
tags: [Vuex2,Vue,Axios]
thumbnail: http://okkula0y9.bkt.clouddn.com/201602011.jpg
---



> 总结笔者最近在项目中使用Vuex2和Axios的总结和小技巧，方便日后查看。本人能力有限，如有错误还请大大们指正探讨😭。

# 阅读前须知
接上一篇的[Axios的配置][1]，阅读本文默认您已经掌握了Vuex2的基本语法、Axios的使用配置与方法、Promise的基本语法。（下面将放出参考链接方便大家在有不明白的地方查看）此篇主要叙述我在实际项目过程中Vuex2和Axios的组合操作。

参考：
- [Vue单向数据流-Vuex2][2]
- [Ajax库-Axios][3]
- [Promise介绍][4]


## Vuex2中的Actions

在Vuex1中actions都可以统一使用`dispatch`去触发`mutation`，那时候去异步这种数据流没什么认识。在Vuex2中规定了,同步的方法使用`commit`去提交`mutation`，异步的方法（如接口请求，完成接口请求后的操作）只能通过分发action去触发，即使用`dispatch`。

![此处输入图片的描述][5]

这里不经吐槽2句，Vuex2的Api还是蛮人性化的，对比Redux为了用好单向数据流，不得不引入一大堆诸如`react-redux`、`redux-saga`、`redux-thunk`...，对新上手的小白来说简直是一脸蒙蔽。上面这些库所带的功能，用Vuex2都可以完成（手动骄傲），当然Vuex2也是站在这些巨人的肩膀上。


下面来看下`vuex2`中关于actions的API
``` javascript

类型: { [type: string]: Function }

在 store 上注册 action。处理函数接受一个 context 对象，包含以下属性：

{
  state,     // 等同于 store.state, 若在模块中则为局部状态
  rootState, // 等同于 store.state, 只存在于模块中
  commit,    // 等同于 store.commit
  dispatch,  // 等同于 store.dispatch
  getters    // 等同于 store.getters
}
```


## Axios的配置

这个是我项目内Axios的配置，主要做了下面几件事情
1. 5秒的超时验证
2. POST的设置
3. 统一的response封装。我通过接口传回的CODE码去判断该请求是否正确
如果想去看更详细的解析可以通过[Axios的配置][1]

``` javascript

import axios from 'axios'
import qs from 'qs'
import * as _ from './whole'    //alert

axios.defaults.timeout = 5000;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8';
axios.defaults.baseURL = 'http://xxx.xxx.com';


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

## 完成一个请求操作

下面介绍一个登陆action的操作，
1. 在接口发起请求时跳出loading
2. 接口请求成功结束loading，并保存数据进入store
3. 接口请求失败抛出异常，结束loading。
4. 返回一个Promise对象，方便继续操作。

``` javascript
export const login = ({ commit },params) => {
  _.busy();               //loding开始
  return new Promise((resolve, reject)=> { 
    axios({
      method:'get',
      url: 'teacher/login',
      params: {
        moblie:params.mobile,
        password:params.password,
      }
    })
    .then((response) => {
        commit(types.LOGIN,response.data.data); //获得的数据通过mutation，存入store中
        _.leave();  //loding结束
        resolve(response);
    })
    .catch((error) => {
      _.leave();  //loding结束
    })
  });
}

const mutations = {
  [types.LOGIN](state, data){
    state.token = data.token;
  }
}

```



``` javascript
//调用action

import { mapActions,mapGetters  } from 'vuex'

export default {
  methods:{
    ...mapActions(['login']),                   //注入action
    _login(){
      let params = {
        mobile:this.mobile,
        pwd:this.password
      }
      this.login(params).
      then(()=>{
          this.$router.replace('/main/index');          //正确完成后进入主页
      })
      .catch((error)=>{                                 //错误则清空密码文本框
          //可以在这里对Error进行捕获，然后根据错误类型进行相对应的处理
          this.pwd = '';                           
      });
    }
  }

```

## 一点说明
因为在Vuex2中action可以直接拿到Store的值，所以我们可以通过 `rootState`这个参数拿到根的store值，通过`state`拿到当前模块中store的值
都没必要在从getter里拿到值，然后通过params的方式传入。

``` javascript

export const xxx = ({state,rootState,commit}) => {
  return new Promise((resolve, reject)=> { 
    axios({
      method:'get',
      url: 'teacher/workbook/class/exercise',
      params: {
        "token":rootState.login.token,          //从store根中拿到数据
        "classCode":state.code,                 //从当前模块中拿到数据
        "chapterId":rootState.route.params.chapterId
      }
    })
    .then((response) => {
      commit(types.WORKBOOK_CLASS_EXERCISE,response.data.data)
      resolve(response);
    })
  });
}

```
这样是不是更方便我们去审查action的代码？



  [1]: https://ygxdxx.coding.me/2017/01/29/Axios%E7%9A%84%E9%85%8D%E7%BD%AE/
  [2]: https://vuex.vuejs.org/zh-cn/index.html
  [3]: https://github.com/mzabriskie/axioss://vuex.vuejs.org/zh-cn/index.html
  [4]: http://www.jianshu.com/p/063f7e490e9aoss://vuex.vuejs.org/zh-cn/index.html
  [5]: https://vuex.vuejs.org/zh-cn/images/flow.png
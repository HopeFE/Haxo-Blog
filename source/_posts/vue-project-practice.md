---
layout: _posts
title: 'Vue小项目的最佳实践'
date: 2017-04-13 13:36:52
tags: Vue
thumbnail: http://okkula0y9.bkt.clouddn.com/2017_4_13.jpg?imageView2/0/q/75|imageslim
---

# Vue小项目的最佳实践


## 项目简介
 目前一期只是为App内某个模块资讯模块文章的分享和APP下载，后续还会有更多的功能,为了项目可扩展、可伸缩结合了我以前的实践搭建了此项目[项目地址][1],如果这个简单的项目能给您带来帮助请给小哥哥⭐Star好不好（手动笔芯）。

### 功能
- 根据分享出来的文章ID获取数据
- 在网页内可以打开或者下载该APP
- 微信平台的特殊处理
    - 微信平台的屏蔽了scheme,文章页面的打开APP的功能需要出浮窗提示去浏览器中打开
    - 下载APP页面在微信中，IOS可以唤起APP Store,安卓则需要提示浮窗
  
  
## 使用技术须知
`Vue`,`VueRouter`,`Vuex`三件套不在多说

### Axios
主要用来发起Http请求，想要详细了解具体使用方式和操作指南可以请参考笔者下面的几篇文章
[Axios全攻略][2]
[Vuex2和Axios的开发][3]
注意：因为`Axios`使用了`Promise`,适配低版本浏览器 一定要配合使用`es6-promise`

### Vuex-router-sync 
[Vuex-router-sync资料][4]
功能：将Router中的 这些数据注入到Store中，方便我们调用。
在此项目中 我用此插件获取URL上的文章ID

### Vue-meta
[Vue-meta资料][5]
功能：改变网页Head上的一些标签值。
在此项目中，我用此插件改变文章页面上的Title，在浏览器中标题不那么木讷。

### Mobi.css
[Mobi.css资料][6]
功能：小而精美的手机端CSS布局库
在此项目中，不想用太大的UI框架也不想自己写太多的样式，选择了它。

### Vue-infinite-loading
[Vue-infinite-loading资料][7]
功能：缓解加载数据时页面空白的尴尬，可自定义loading动画。

## 关键实现

### URL上获取文章ID
和APP的同学商量了我们就用http://xxxx/article/:id的方式去定义分享出去文章地址，页面通过获得URL上的
ID去请求相对应的数据。我是使用`Vuex-router-sync`直接从Store中获得ID的`rootState.route.params.id`
```
// 文件src/modules/action
/** 获取文章信息 */
export const getArticle = ({ rootState, commit }) => {
  return new Promise((resolve, reject) => {
    axios({
      method: 'get',
      url: 'share/news_details',
      params: {
        news_id: rootState.route.params.id
      }
    })
    .then((response) => {
      commit(types.ARTICLE, response.data.data)
      resolve(response)
    })
    .catch((error) => {
      reject(error)
    })
  })
}
```
### Store中获取环境区别
在每个页面进行操作时，我们需要鉴别当前系统是IOS或者安卓，每次通过正则去鉴别UA里的字符串太麻烦，所以我将此放到Store中，方便所有的组件使用

```
// 文件src/modules/store
const state = {
  system: (/iphone|ipad|ipod/.test(navigator.userAgent.toLowerCase()) ? 'IOS' : 'Android'),
  article: {},
  isWeixin: (/MicroMessenger/ig).test(navigator.userAgent)
}
```
### 合理的处置异常
我们去加载数据时可能会遇到失败的情况，这里需要对页面有一个良好的处理，这里我主要使用`Vue-infinite-loading`去实现页面上的效果。
``` 
_onInfinite () {
  this.getArticle().then(() => {
    // 完成之后loading消失 
    this.$refs.infiniteLoading.$emit('$InfiniteLoading:loaded')
    this.$refs.infiniteLoading.$emit('$InfiniteLoading:complete')
  })
  .catch(() => {
    // 异常之后页面的展示  执行下方slot="no-results"部分
    this.$refs.infiniteLoading.$emit('$InfiniteLoading:complete')
  })
}
```

``` 
<infinite-loading :on-infinite="_onInfinite" ref="infiniteLoading" spinner="bubbles">
  <span slot="no-results">好像来到了奇怪的地方~</span>
  <span slot="no-more"></span>
</infinite-loading>
```

### keep-alive组件复用
这是一个很能提高页面性能的标签，会将已使用过的不活动的组件缓存起来而不是销毁。在性能不太好的手机上，模版的渲染也是需要一定时间的，我们可以用这个标签将缓存曾经使用过的组件（页面），在此组件激活时刷新里面的数据即可。激活时使用[activated][8]这个生命周期
![activated][9]

```
  activated () {
    this.clearArticle() //激活时先清除Store中的数据 因为$InfiniteLoading是根据页面高度来发起请求的
    this.$refs.infiniteLoading.$emit('$InfiniteLoading:reset')
  }
```

### 组件代码的规范
> 始终基于模块的方式来构建你的 app，每一个子模块只做一件事情。
Vue.js 的设计初衷就是帮助开发者更好的开发界面模块。一个模块是应用程序中独立的一个部分。

我们需要将我们`*.vue`文件按照一定的结构组织，使得组件便于理解，主要有以下几点比较重要：

- 导出一个清晰、组织有序的组件，使得代码易于阅读和理解。同时也便于标准化。
- 按首字母排序属性，data, computed, watches 和 methods 使得属性便于查找。
- 合理组织，使得组件易于阅读。(name; extends; props, data and computed; components; watch and methods; lifecycle methods, 等.);
- 使用 name 属性。借助于vue devtools可以让你更方便的测试
- 合理的 CSS 结构，如 BEM 或 rscss - 详情?;
- 使用单文件 .vue 文件格式来组件代码

同时配合`ESLint`将代码写的更加规范和阅读，我这边使用`Standard`的风格，在VScode中也开启了Standard的验证。
[ESLint官网][10]
[JavaScript 代码规范-Standard风格][11]
组件规范也可以参考笔者这篇：[Vue.js 组件编码规范][12]

## 最后
看起来此项目简单，实则上用了不少插件去实现需要较强的动手（第三方坑也多，选择一个好的插件得先去github上看看，作者的代码质量），需要保持一定的弹性方便日后的扩展也要避免过度的设计。大家若想要加速自己的开发速度，可以多逛逛[Vue awesome][13]上看看大多数都是高质量的插件，其实很多轮子都有人造好了，选取好的直接拿来用岂不妙哉？


  [1]: https://github.com/HopeFE/Ant_App_H5
  [2]: https://blog.ygxdxx.com/2017/02/27/Axios-Strategy/
  [3]: https://blog.ygxdxx.com/2017/02/01/Vuex2&Axios-Develop/
  [4]: https://github.com/vuejs/vuex-router-sync
  [5]: https://github.com/declandewet/vue-meta.org/zh-cn/index.html
  [6]: https://peachscript.github.io/vue-infinite-loading
  [7]: https://peachscript.github.io/vue-infinite-loading
  [8]: https://vuefe.cn/v2/api/#activated
  [9]: http://okkula0y9.bkt.clouddn.com/20170413.jpg
  [10]: http://eslint.org/
  [11]: https://github.com/feross/standard/blob/master/docs/README-zhcn.md
  [12]: https://blog.ygxdxx.com/2017/03/09/Vuejs-Component-Style-Guide/
  [13]: https://github.com/vuejs/awesome-vue#libraries--plugins
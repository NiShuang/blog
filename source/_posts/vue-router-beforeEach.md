---
title: 解决刷新页面不触发 vue-router 的 beforeEach 钩子的问题
date: 2017-06-30 19:12:33
tags:
 - Vue
 - js
categories: 前端笔记
---

### 问题

在使用 Vue.js 开发项目时，会用到 vue-router 模块来进行路由管理。为了在用户访问每个页面之前判断用户是否有访问该页面权限，需要用到 vue-router 的 beforeEach 全局钩子，在这个钩子中进行权限判断，决定允许或拒绝用户访问，或者是跳转到登录界面。

代码如下(vue-router 1.0)：

``` javascript
Vue.use(Router)

// routing
var router = new Router()

router.map({
  '/reject': {
    component: RejectView
  },
  '/login': {
    component: LoginView
  }
})

router.start(App, '#app')

router.beforeEach(function (transition) {
  if ( transition.to.path.indexOf('/login') === 0) {
    transition.next()
    return
  }
  if (document.cookie.indexOf('isLogin=true') < 0) {
    router.go('/login?redirect=' + transition.to.path)
    transition.next()
  } else {
    var cname = transition.to.params.cname
    if (cname !== undefined) {
      var has_permission = store.state.userInfo.power[Nav[cname]]
      if (!has_permission) {
        router.go('/reject')
        transition.next()
        return
      }
    }
    transition.next()
  }
})
```

然而出现一个bug，当我手动 F5 刷新页面时，却没有触发 beforeEach 钩子。
<!-- more --> 

### 原因

查询 vue-router1.0 的说明文档，文档上的 Basic Usage 代码如下：

``` javascript
// Load the plugin
Vue.use(VueRouter)

// Define some components
var Foo = {
    template: '<p>This is foo!</p>'
}

var Bar = {
    template: '<p>This is bar!</p>'
}

// The router needs a root component to render.
// For demo purposes, we will just use an empty one
// because we are using the HTML as the app template.
// !! Note that the App is not a Vue instance.
var App = {}

// Create a router instance.
// You can pass in additional options here, but let's
// keep it simple for now.
var router = new VueRouter()

// Define some routes.
// Each route should map to a component. The "component" can
// either be an actual component constructor created via
// Vue.extend(), or just a component options object.
// We'll talk about nested routes later.
router.map({
    '/foo': {
        component: Foo
    },
    '/bar': {
        component: Bar
    }
})

// Now we can start the app!
// The router will create an instance of App and mount to
// the element matching the selector #app.
router.start(App, '#app')
```

可以发现，样例中把 router.start(App, '#app') 这行代码放在了最后。因为这一步是创建和挂载根实例，是启动路由的最后一步，需要在定义路由实例和配置路由之后进行。

### 解决

修改以后的代码如下：

``` javascript
Vue.use(Router)

// routing
var router = new Router()

router.map({
  '/reject': {
    component: RejectView
  },
  '/login': {
    component: LoginView
  }
})


router.beforeEach(function (transition) {
  if ( transition.to.path.indexOf('/login') === 0) {
    transition.next()
    return
  }
  if (document.cookie.indexOf('isLogin=true') < 0) {
    router.go('/login?redirect=' + transition.to.path)
    transition.next()
  } else {
    var cname = transition.to.params.cname
    if (cname !== undefined) {
      var has_permission = store.state.userInfo.power[Nav[cname]]
      if (!has_permission) {
        router.go('/reject')
        transition.next()
        return
      }
    }
    transition.next()
  }
})

router.start(App, '#app')
```

### 总结

在使用 vue-router 模块时，挂载根实例的步骤要放在最后，不然会导致配置不成功。

### 参考资料

 - [vue-router 2 中文文档](https://router.vuejs.org/zh-cn/)

---

> 文章标题：[解决刷新页面不触发 vue-router 的 beforeEach 钩子的问题](http://www.cielni.com/2017/06/30/vue-router-beforeEach/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2017/06/30/vue-router-beforeEach/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！

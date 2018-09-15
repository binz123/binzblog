---
layout: post
title:一步一步来的Vue-router源码解析
categories: javascript,Vue,Vue-router
description: 对Vue-router源码的解析跟自我提升
keywords:Vue VueRouter 源码
---
## 前言
最近跟大佬聊天，说了一下自己的技术，能力跟将来的目标，大佬听到我说用Vue比较多就问我有没有看过Vue-router的源码。当时有点懵，Vue-router的源码？Vue的源码我倒是看过一点，说起来也能头头是道，但Vue-router的源码，Umm...它不就是利用了hash跟组件间的对应关系且对URL中hash的改变并不会刷新页面实现的么？然后大佬就问，hash值的改变让组件更新是怎么实现的呢？ 
   
彻底蒙圈了，说实话我还真一直没把这当一回事，现在才发现自己格局不够，赶紧回来怒刷了一波源码。断断续续的也是算是摸到了一点Vue-router的门道，先一步一个脚印的记录下来吧。

## Vue-router
在Vue中，所有的内容都可以说是组件化的，我们通过路径与组件进行对应，由此将页面渲染出来。  
### 1.创建一个路由
```javascript
const routes = [
    {
    name:'/',
    component:()=>import('../home/index.vue')
    },
    {
    name:'/other',
    component:()=>import('../other/other.vue')
    }
]
``` 
### 2.创建一个router对象，对routes进行管理
```javascript
//利用构造函数VueRouter,传入构造函数routes
const router = new VueRouter({routes});

```
### 3.将router实例注入到Vue中
```javascript
new Vue({
    router,
})
```

以**router-link**为例，当我们点击router-link时候，to属性将于我们配置{path:'/...',component:()=>import('../xxx')}进行对应，并通过相应的组件完成对页面的渲染

## 路由模式
根据Vue-router的官方文档可以得知，目前常用的路由有以下两种模式：  
**Hash**  
**History**  
  
实际上除了这两种模式，Vue-router还提供了一种**Abstract**模式，这个主要用于非浏览器模式下，这里就不多描述了。  
并且我们可以得知： 
   
Hash模式利用的是*URL的Hash*来模拟一个完整的URL，当URL发生改变的时，页面并不会重载。  
History模式则利用H5中的*history.pushState* API来完成URL跳转而无需加载页面。


## 源码目录结构

![vuerouter](http://binzhome.com/assets/images/aboutvuerouter/directory.jpg)


- components文件夹下是router-view,router-link两个组件的定义
- history文件夹下分别对应三种路由的封装方式
- util下为一些用到的功能函数的整合
- create-matcher.js，create-route-map.js就是对应的路有生成匹配表了，我们用到match方法就在这里面
- indx是VueRouter的定义
- install中包含了VueRouter的包装方法以及如何映射到Vue实例的方法

## VueRouter
首先我们来看看VueRouter的定义：
```javascript
export default class VueRouter {
  static install: () => void;
  static version: string;

  ...

  constructor(options: RouterOptions = {}) {
    this.app = null
    this.apps = []
    this.options = options
    this.beforeHooks = []
    this.resolveHooks = []
    this.afterHooks = []
    this.matcher = createMatcher(options.routes || [], this)

    //根据传入的options选择相应的路由模式，默认为hash
    let mode = options.mode || 'hash'
    //如果传入的是history模式，则根据判断当前是否支持该模式，supportsPushState在工具函数中用于判断当前浏览器是否支持hash模式
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    //若不支持history则改为hash
    if (this.fallback) {
      mode = 'hash'
    }
    //判断是否在浏览器下，若不是则为abstract模式
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode

    //根据不同的路由模式实例化对应的history对象
    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this, options.base)
        break
      default:
        if (process.env.NODE_ENV !== 'production') {
          assert(false, `invalid mode: ${mode}`)
        }
    }
  }


```
从上面的代码中我们可以看出VueRouter的实例主要干了以下几件事：  
1.对options进行了一个赋值。  
2.根据当前传入的模式以及执行环境，判断是使用hash，history还算是abstract模式。  
3.根据不同的模式，实例化对应的history对象，并将其作为VueRouter的history属性。

接下来我们再来看VueRouter中init方法，这个方法中进行了路由的初始化，并且进行了非常重要的的一步：***监听Hash的变化***  
```javascript
//路由初始化
  init(app: any /* Vue component instance */) {

    ...

    this.apps.push(app)

    // main app already initialized.
    if (this.app) {
      return
    }

    this.app = app

    const history = this.history

    //根据history的类型分别进行不同的处理
    if (history instanceof HTML5History) {
    //若为history模式，则直接返回当前URL进行匹配
      history.transitionTo(history.getCurrentLocation())
    } else if (history instanceof HashHistory) {
      //若为hash模式，则返回当前URL进行匹配并设置对Hash的监听
      //注意：这里开始设置对hash变化的监听！！
      const setupHashListener = () => {
        history.setupListeners()
      }
      history.transitionTo(
        history.getCurrentLocation(),
        setupHashListener,
        setupHashListener
      )
    }
    
    history.listen(route => {
      this.apps.forEach((app) => {
        app._route = route
      })
    })
  }
```
在VueRouter类中还定义了诸如导航守卫，push，replace方法等,并且可以看出诸如push,replace等这些方法，其实VueRouter只是做了一层封装，它其实是调用的是history中对应的方法
```javascript
  beforeEach(fn: Function): Function {
    return registerHook(this.beforeHooks, fn)
  }

  beforeResolve(fn: Function): Function {
    return registerHook(this.resolveHooks, fn)
  }

  afterEach(fn: Function): Function {
    return registerHook(this.afterHooks, fn)
  }

  ...

  push(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    this.history.push(location, onComplete, onAbort)
  }

  replace(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    this.history.replace(location, onComplete, onAbort)
  }

  go(n: number) {
    this.history.go(n)
  }

  back() {
    this.go(-1)
  }
```
## history
看完了上面的源码是不是还是一头雾水，说实话我看完也是一头雾水，想解决这些雾水就必须解决下面两个问题。  
1.```transitionTo()```方法有什么用？  
2.history到底是干什么的，它帮VueRouter做了什么？  
3.```setupListeners()```方法是如何设置监听的？

带着这三个疑问我们开始看history中的文件，base.js文件中主要定义History类，而HTML5History，HashHistory，AbstractHistory都是继承自base.js中定义的History。  

```javascript
export class History {
  
  ...

constructor(router: Router, base: ?string) {
  this.router = router
  this.base = normalizeBase(base)
   
   ...
}

listen(cb: Function) {
  this.cb = cb
}


//用来处理路由变换中的逻辑及回调，划重点
transitionTo(location: RawLocation, onComplete ?: Function, onAbort ?: Function) {
  //1.根据传入的值与当前值进行对比，并返回相应的路由对象
  const route = this.router.match(location, this.current)
  //2.判断新路由是否有效，是否与当前路由相同
  this.confirmTransition(route, () => {
    //3.此处更新路由对象，并对视图进行更新 重点
    this.updateRoute(route)
    //4.若存在Complete回调函数则调用回掉函数
    onComplete && onComplete(route)
    this.ensureURL()

    // fire ready cbs once
    if (!this.ready) {
      this.ready = true
      this.readyCbs.forEach(cb => { cb(route) })
    }
  }, err => {
    if (onAbort) {
      onAbort(err)
    }
    if (err && !this.ready) {
      this.ready = true
      this.readyErrorCbs.forEach(cb => { cb(err) })
    }
  })
}

...

updateRoute(route: Route) {
  const prev = this.current
  this.current = route
  //之前listen中设置的cb
  this.cb && this.cb(route)
  //调用afterEach钩子函数
  this.router.afterHooks.forEach(hook => {
    hook && hook(route, prev)
  })
}

}
```
而且看出，当传入一个新的路由的时候会transitionTo()会进行如下处理:  
1.根据传入路由与当前路由进行对比，并返回相应的路由对象  
2.判断新路由是否有效，并分别调用对应的回调函数
3.若有效，则调用成功回掉，结合初始化中
```javascript
 const setupHashListener = () => {
        history.setupListeners()
      }
      history.transitionTo(
        history.getCurrentLocation(),
        setupHashListener,
        setupHashListener
      )
```
到这里，我们就理来了Hash模式下整个设置路由监听函数的过程了。  
这里有一个非常重要的地方：***this.update(route)***
在这里我们调用了update(route) ，并且在这里面调用this.cb，这个cb就是我们listen中设置的cb，那这个listen（）方法又是在哪里用的？回到index.js中的init()方法，我们会发现一小段如下的代码：
```javascript
 history.listen(route => {
      this.apps.forEach((app) => {
        app._route = route
      })
    })
```
哦！！原来我们将改变后的路由绑定到了Vue实例的_route属性上了，如果_route属性变化了根据Vue响应式原理是不是就要触发render了呢？没错，就是这样的。至于VueRouter的router是如何与Vue的_router联系的上的，我们打开install.js可以看到下面熟悉的代码：
```javascript
  Vue.mixin({
    beforeCreate () {
      if (isDef(this.$options.router)) {
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })
```
在这里我们用mixin方法在每个Vue实例beforeCreate的时候通过Vue.util.defineReactive(this, '_route', this._router.history.current)，将这个属性加到对应的Vue实例中，它的方法就像Vue中data绑定到实例上一样的。  

这样一来，一旦我们调用updateRoute发生变化，通过双向绑定的原理我们就实现了updateRoute-->render的过程了。虽然这段代码又长又绕，我个人认为这里就是Vue-router最漂亮的地方，你中有我，我中有你。  
  
好了，到这里我们就解释完了transitionTo到底拿来做什么的了，总的来说，它就是路由变化需要进行的一系列逻辑操作与回调的结合。
并且我们也基本可以看出来Hash改变与视图变化的关联了。
HashChange->triggerListener->transitionTo()->confirmTransitionTo()->updateRoute()->app._route=newroute->render  

我们再细点到具体的History类型

### HashHistory
```javascript
export class HashHistory extends History {
  constructor(router: Router, base: ?string, fallback: boolean) {
    super(router, base)
    //判断是否是从history模式降级的，若是则返回
    if (fallback && checkFallback(this.base)) {
      return
    }
    ensureSlash()
  }

  //设置对window对象hashchange事件的监听将在mounted以后进行，避免了监听过早无效
  setupListeners() {
    const router = this.router
    const expectScroll = router.options.scrollBehavior
    const supportsScroll = supportsPushState && expectScroll

    if (supportsScroll) {
      setupScroll()
    }

    window.addEventListener(supportsPushState ? 'popstate' : 'hashchange', () => {
      const current = this.current
      //此处判断是否hash值是否为/开头，若不是则需要对其进行替换一次
      if (!ensureSlash()) {
        return
      }
      //进行路由改变后的逻辑处理
      this.transitionTo(getHash(), route => {
        if (supportsScroll) {
          handleScroll(this.router, route, current, true)
        }
        if (!supportsPushState) {
          replaceHash(route.fullPath)
        }
      })
    })
  }

//router.push()对应的方法
  push(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      pushHash(route.fullPath)
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }

//router.replace对应的方法
  replace(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      replaceHash(route.fullPath)
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }

  go(n: number) {
    window.history.go(n)
  }

  ensureURL(push?: boolean) {
    const current = this.current.fullPath
    if (getHash() !== current) {
      push ? pushHash(current) : replaceHash(current)
    }
  }

  getCurrentLocation() {
    return getHash()
  }
}


//获取hash值
export function getHash(): string {
  //这里之所以用window.location.href而不用window.location.hash的原因：
  //在火狐下将会被预编译
  const href = window.location.href
  const index = href.indexOf('#')
  return index === -1 ? '' : decodeURI(href.slice(index + 1))
}

function getUrl(path) {
  const href = window.location.href
  const i = href.indexOf('#')
  const base = i >= 0 ? href.slice(0, i) : href
  return `${base}#${path}`
}

function pushHash(path) {
  if (supportsPushState) {
    pushState(getUrl(path))
  } else {
    window.location.hash = path
  }
}

function replaceHash(path) {
  if (supportsPushState) {
    replaceState(getUrl(path))
  } else {
    window.location.replace(getUrl(path))
  }
}

...
```

其实HashHistory与HTML5History中的代码其实就短短100多行，我也尽量减少了部分代码，但也保留了我们常用到一些操作。  
现在就让我们看看setupListeners到底是如何监听浏览器Hash变化的。  
  
首先，我们会先判断当前的模式是否为history的降级，若是我们将使用popState进行事件的监听，若不是我们则用hashchange事件，至于回调函数我们使用的就是transitionTo()方法，短短的几行字貌似就解答完了setupListeners，是的，它确实就只干了这么多活。  
  

当然了，这里有一些细节的问题还是有必要提一下。一个是getHash()方法，这里之所以用window.location.href而不用window.location.hash的原因是location.hash在火狐浏览器下存在兼容性问题，所以我们直接获取整个URL并在进行二次加工得到hash的值。  
二是pushHash与replaceHash的区别，push方法只会修改url后面hash的值，每一次改变hash(window.location.hash)，都会在浏览器访问历史中增加一个记录。而replace则会将整个url进行替换，push方法会在浏览器历史访问栈中添加一个新的记录，而replace则是强制替换了整个url，这样做主要是用于用户首次跳入切当Hash值不为'/'的时候进行直接替换。

那么除了监听外的push，replace方法的实现呢？这里就不用废话了，其实是它跟监听回调函数里对应的方法是一样的。

### HTML5History
HTML5History
```javascript
//通过添加popstate进行监听
window.addEventListener('popstate', e => {
      const current = this.current

      ...

      this.transitionTo(location, route => {
        if (supportsScroll) {
          handleScroll(router, route, current, true)
        }
      })
})

    push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      pushState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }

  replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      replaceState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }
```
这里我们主要是通过window.onpopstate与history.pushState(),history.replaceState()结合完成的。这两个方法都有一个共同点：  
调用这两个方法修改了浏览器的历史栈以后，虽然url发生了改变，但是浏览器并不会马上发送请求该url，这也是为什么前端单页面应用不用重新发送请求的前提了。

### AbstractHistory
AbstractHistory并不涉及和浏览器地址有关的记录，其流程与HashHistory一样，但是却是通过数组抽象了浏览器历史栈的功能。
```javascript
export class AbstractHistory extends History {
  index: number;
  //抽象成浏览器历史栈的功能。
  stack: Array<Route>;

  constructor (router: Router, base: ?string) {
    super(router, base)
    this.stack = []
    this.index = -1
  }
  ...
  }
```

## 总结
好了，到这里Vue-router的基本原理跟实现大致已经讲清楚了，现在来做个总结吧。  
```
1.new VueRouter(options)创建了一个新的router对象 

2.通过Vue.mixin()在所有Vue实例beforeCreate的时候讲route映射到对应实例的_router属性上，并对该属性进行双向绑定 

3.VueRouter进行初始化，并根据执行环境与mode类型生成对应的History对象。并根据模式设置首次监听回调函数，若‘history’则监听popState，若‘hash‘则监听hashChange，设置history.listen的回调函数

4.当push,replace或其他方式对url进行修改时，调用History对象的transitionTo()方法，在这个方法中判断是否符合更新路有的逻辑，若更新路由，则通过updateRoute()调用history.listen中的回调函数将app._route更新为新的路由

5.此时app._route的值发生了改变，自动调用Vue实例的render()方法
```
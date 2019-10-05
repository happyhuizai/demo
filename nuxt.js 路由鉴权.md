## nuxt.js 路由鉴权

1. 中间件authenticated.js判断是否有权限

```js
import Cookie from 'js-cookie'
 
export default function ({store, route, redirect, req}) {
  const {auth} = Cookie.get(req)
  if (auth) {
    store.commit('setAuth', auth)
    return store.dispatch('getUserInfo')
  }
  const routePath = route.path
  const extraPath = ['/']// 白名单
  if ((!store.state.auth) && extraPath.indexOf(routePath) === -1) {
    // 跳转到登录页面
    return redirect('/')
  }
}
```

2. 在nuxt.config.js 中使用中间件

```js
router: {
  middleware: ['authenticated'],
}
```

3. 添加nuxtServerInit函数,因为nuxtServerInit实在服务端执行的所以要在req获取cookie

```js
nuxtServerInit({commit, state}, {req}) {
  let auth = null
  if (req.headers.cookie) {
    const parsed = cookieparser.parse(req.headers.cookie)
    try {
      auth = JSON.parse(parsed.auth)
    } catch (err) {

    }
    commit('setAuth', auth)
  }
}

```

4. axios 拦截器添加token到header中

```js
let token = Cookie.get('auth')
  if (token) {
    config.headers = {
      'Authorization': `Token ${token}`
    }
  }
```


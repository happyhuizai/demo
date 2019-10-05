## vue权限路由实现方式

### 实现过程


>应用初始化的时候只挂载不需要权限控制的路由

```js
const constRouterMap = [{
        name: "login",
        path: "/login",
        meta: {
            layout: true
        },
        component: () => import("@/views/Login.vue")
    },
    {
        path: "/404",
        meta: {
            layout: true
        },
        component: () => import("@/views/Page404.vue")
    },
    {
        path: "/init",
        name:'init',
        meta: {
            layout: true
        },
        component: () => import("@/views/Init.vue")
    },
    {
        path: "*",
        meta: {
            layout: true
        },
        redirect: "/404"
    }
];
export default constRouterMap;
```

```js
import Vue from "vue";
import Router from "vue-router";
import constRouterMap from "./router/constRouterMap";

Vue.use(Router);

export default new Router({
  mode: 'history',
  scrollBehavior: () => ({ y: 0 }),
  routes: constRouterMap
});
```

>登录成功后跳到404页面然后跳转到登录页面

```js
export default {
  name: "page404",
  mounted() {
    if (!this.$store.state.isLogin) {
      this.$router.replace({ path: "/login" });
      return;
    }
    if (!this.$store.state.initedApp) {
      this.$router.replace({ path: "/init" });
      return;
    }
  }
};
</script>
```

>登录成功后跳到/路由

```js
export default {
  name: "login",
  data() {},
  computed: {
    logining() {
      return this.$store.state.isLogin;
    }
  },
  beforeRouteEnter (to, from, next) {
    if (window.localStorage.getItem("TOKEN")) {
      next(from)
    }else{
      next()
    }
  },
  methods: {
    submit() {
      this.$refs.form.validate(async () => {
        const data = await this.$store.dispatch("login", { ...this.form });
        if (data) {
          this.$router.push(this.$route.query.redirect || '/')
          return false;
        } else {
          return false;
        }
      });
    }
  }
};
```

>因为当前没有/路由，会跳到/404 根据判断已经登陆成功会跳转到init页面
init组件里判断应用是否已经初始化(避免初始化后，直接从地址栏输入地址再次进入当前组件)。
如果已经初始化，跳转/路由(如果后端返回的路由里没有定义次路由，则会跳转404)。
没有初始化，则调用远程接口获取菜单和路由等，然后处理后端返回的路由，将component赋值为真正
的组件，接着调用addRoutes挂载新路由，最后跳转/路由即可。

```js
<template>
  <div></div>
</template>
<script>
import components from "../router/routerComponents.js";

export default {
  name: "init",
  async mounted() {
    if (!this.$store.state.isLogin) {
      this.$router.push({ path: "/login" });
      return;
    }
    if (!this.$store.state.initedApp) {
      const loading = this.$loading({
        lock: true,
        text: "初始化中",
        spinner: "el-icon-loading",
        background: "rgba(0, 0, 0, 0.7)"
      });
      let data = await this.$store.dispatch("initAuth");
      let menus = data.data;
      var routers = [...menus];
      for (let router of routers) {
        let component = components[router.component];
        router.component = component;
      }
      this.$router.addRoutes(routers);
      loading.close();
      this.$router.replace({
        path: "/"
      });
      return;
    } else {
      loading.close();
      this.$router.replace({
        path: "/"
      });
    }
  }
};
</script>
```

>以上实现方式好处就就是不使用全局路由守卫


[具体实现请看](https://github.com/happyhuizai/element-admin)

参考文章

https://github.com/PanJiaChen/vue-element-admin

https://juejin.im/post/5b5bfd5b6fb9a04fdd7d687a#heading-15

http://refined-x.com/2017/09/01/用addRoutes实现动态路由/

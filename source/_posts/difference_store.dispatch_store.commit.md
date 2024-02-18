---
title: The difference between this.\$store.dispatch() and this.\$store.commit() in vuex
date: 2021-03-16 10:42:10
tags: 
	- vue
	- vuex
categories: ['vue']
id: vuexStoreDispatchAndCommit
---
# Difference

The difference between `this.$store.dispatch()` and `this.$store.commit()` methods is that in general they are only different in **the way values are accessed**, both methods pass the value to vuex's mutation to change the state.

- `this.$store.dispatch()`: contains asynchronous operations, such as submitting data to the background, written: `this.$store.dispatch('action method name', value)`
- `this.$store.commit()`: synchronous operation, written: `this.$store.commit('mutations method name', value)`

<!-- more -->

>  commit: Synchronize operations
>
> - Storage value: `this.$store.commit('changeValue',name)`
> - Fetch value:  `this.$store.state.changeValue`

> dispatch: Asynchronous operation
>
> - Storage value: `this.$store.dispatch('getlists',name)`
> - Fetch value:  `this.$store.getters.getlists`

# Example

## Vuex file `src/store/index.js`

```js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

export const store = new Vuex.Store({
  // state专门用来保存共享的状态值
  state: {
    // 保存登录状态
    login: false
  },
  // mutations: 专门书写方法,用来更新 state 中的值
  mutations: {
    // 登录
    doLogin(state) {
      state.login = true;
    },
    // 退出
    doLogout(state) {
      state.login = false;
    }
  }
});
```

## JS Component section: `src/components/Header.vue`

```javascript
<script>
// 使用vux的 mapState需要引入
import { mapState } from "vuex";

export default {
  // 官方推荐: 给组件起个名字, 便于报错时的提示
  name: "Header",
  // 引入vuex 的 store 中的state值, 必须在计算属性中书写!
  computed: {
    // mapState辅助函数, 可以快速引入store中的值
    // 此处的login代表,  store文件中的 state 中的 login, 登录状态
    ...mapState(["login"])
  },
  methods: {
    logout() {
      this.$store.commit("doLogout");
    }
  }
};
</script>
```

## JS Component section: `src/components/Login.vue`

```javascript
<script>
export default {
  name: "Login",
  data() {
    return {
      uname: "",
      upwd: ""
    };
  },
  methods: {
    doLogin() {
      console.log(this.uname, this.upwd);
      let data={
        uname:this.uname,
        upwd:this.upwd
      }
       this.axios
        .post("user_login.php", data)
        .then(res => {
          console.log(res);
          let code = res.data.code;

          if (code == 1) {
            alert("恭喜您, 登录成功! 即将跳转到首页");

            // 路由跳转指定页面
            this.$router.push({ path: "/" });

            //更新 vuex 的 state的值, 必须通过 mutations 提供的方法才可以
            // 通过 commit('方法名') 就可以触发 mutations 中的指定方法
            this.$store.commit("doLogin");
          } else {
            alert("很遗憾, 登陆失败!");
          }
        })
        .catch(err => {
          console.error(err);
        });
    }

  }
};
</script>
```


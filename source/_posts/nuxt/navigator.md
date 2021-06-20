---
title: nuxt导航共用
categories: nuxt
tags: [nuxt]
date: 2020-1-20
---  

导航作为多个页面之间共用的部分，一般是动态的，即每个页面返回之前需要异步请求。

服务端异步请求的方法有asyncData，fetch。但是这俩方法都只支持页面组件，不支持layout中使用。

所以，我们只能用vuex作为中转站，将页面组件获取的导航信息存储到vuex中。layout再从vuex中获取。

layout/default
```javascript
import navigator from '@@/navigator/index.vue'
export default {
  name: 'Music163',
  components: { navigator },
  data () {
    return {
    }
  },
  computed: {
    menus () {
      return this.$store.state.menu.menus
    }
  }
}
```

pages/index
```javascript
export default {
  fetch ({ store, params }) {
    store.dispatch('menu/GET_MENU')
  }
}
```

store/menu
```javascript
export const state = () => ({
  menus: []
})

export const mutations = {
  SET_MENU (state, menus) {
    state.menus = menus
  }
}

export const actions = {
  GET_MENU (store, menus) {
    const data = [{
      title: '发现音乐'
    }, {
      title: '我的音乐'
    }]
    store.commit('SET_MENU', data)
  }
}

```
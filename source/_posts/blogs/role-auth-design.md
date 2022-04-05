---
title: 说一说自己设计的运营后台权限
categories: 其它
tags: [权限, 运营后台]
date: 2019-3-2
--- 

## 权限设计

我们的运营后后台的用户由多个角色组成。不同角色的使用权限区别在于：
1. 可见的菜单不同
2. 同一个菜单页面内可见的操作按钮不同，如是否存在编辑按钮
3. 页面内的操作按钮可能对应着其它不在菜单内显示的路由，如编辑页面

根据以上分析，我们可以提炼出3个关键词：
* 菜单
* 路由
* 权限


3者之间的关系我们再捋一捋。
1. 从顶级菜单开始，每个菜单是树状结构，每个叶子节点对应一个路由，这个路由是由1个或多个权限决定的（如查看或编辑权限任意一个存在就决定这个路由的存在）。
2. 除了每个菜单的叶子节点对应一个路由之外，有些路由是隐藏在操作中的，和某些操作权限相关。
3. 有些权限仅控制页面内操作，和任何路由无关。

根据以上分析，可以构思出这3个关键词的结构：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-12/1623467846986-image.png)

根据上图可知，权限可以有2种行为：
* 控制路由
* 控制界面

服务端维护一份菜单，菜单的字段如下：

```
{
    menu_code: '', // 唯一标识
    menu_name: '', // 菜单名称
    p_menu_code: '', // 父级菜单
}
```
以上结构可以维护一份菜单树。

另外维护一份路由，字段如下：

```
{
    route_code: '', // 唯一标识
    route_path: '', // 客户端路由路径
    menu_code: '', // 关联的菜单（为空表示不显示在菜单栏，通过某操作可以跳转）
}
```

另外最重要的是权限，字段如下：
```
{
    auth_code: '', // 唯一标识
    auth_name: '', // 权限名称
    route_codes: [], // 关联的路由列表（若为空，表示该权限只影响界面按钮）
}
```

如果想要在配置的时候通过菜单分类显示权限，就需要通过菜单反查对应权限，因此需要反向维护关联关系，即
* 菜单表要关联路由
* 路由表关联权限

菜单表：
```
{
    menu_code: '', // 唯一标识
    menu_name: '', // 菜单名称
    p_menu_code: '', // 父级菜单,
    relate_route: [] // 关联的路由
}
```
路由表：

```
{
    route_code: '', // 唯一标识
    route_path: '', // 客户端路由路径
    menu_code: '', // 关联的菜单
    relate_auth: [] // 关联的权限列表
}
```

## 实现

### 权限配置
以顶级菜单分类显示可配置的路由：

```
- 用户管理
    —— 用户查看
    —— 用户新增/编辑
- 商品管理
    —— 商品查看
    —— 商品添加/编辑基本信息
    —— 商品价格日历修改
    —— 商品上下架
- 订单管理
    —— 订单查看
    —— 订单操作退款
    —— 财务结算
```

通过菜单的反向的关联关系，可以分类显示所有可配置的权限。
将选中的权限列表提交后，每个角色有一份自己的权限表。

### C端实现

##### 1. 登录接口返回该用户所属角色对应的权限表、路由表，树结构的菜单
    
* 权限表用于界面内操作控制
* 路由表用于注册有权限的路由
* 树结构的菜单用来初始化菜单栏

##### 2. 动态注册路由

路由分为3种，默认路由、重定向路由、受权限控制路由。

```
// 默认路由
export const defaultRootes = [
  {
    path: '/login',
    // component: login
    components: {
      login: () => import(/* webpackChunkName: "base" */'@/views/login')
    }
  },
  {
    path: '/loading',
    // component: loading
    component: () => import(/* webpackChunkName: "base" */'@/components/bodyView/components/loading')
  }
]

// 重定向路由
export const redirectRoutes = [
  {
    path: '/',
    // 默认重定向路由（登录后根据权限修改）
    redirect: '/login'
  },
  {
    path: '*',
    // component: notFound
    component: () => import(/* webpackChunkName: "base" */'@/components/bodyView/components/404')
  }
]

// 受权限控制路由
export const routes = [
  {
    path: '/order/list',
    // component: orderList
    component: () => import(/* webpackChunkName: "order" */'@/views/order')
  },
  ...
]

```

关于重定向，有2个细节需要注意：
1. 根路由'/'的重定向路由是不固定的。
    * 在登录前，'/'的重定向路由是'/login'
    * 成功登录后，需要取出第一个菜单下的第一个路由作为重定向路由。
2. 已登录用户的"/login"路由需要重定向到首页


在App.vue中注册默认路由：

```
created() {
    this.path = this.$route.path
    // 注册基本路由
    this.$router.addRoutes([...defaultRootes, ...redirectRoutes])
    // 检查登录状态
    this.checkLogin()
},
methods: {
    checkLogin() {
      // 通过获取用户信息判断是否已经登录
      apiGetUserInfo().then((response) => {
        /* 已登录 */
        // 1. 保存用户信息 
        // 2. 初始化菜单
        this.menus = response.data.menus
        
        // 3. 注册路由
        const routes = response.data.routes
        // 将根路由'/'的重定向路由改为接口返回的第一个路由：
        redirectRoutes[0].redirect = routes[0].path
        
        const newRouter = new Router({
            mode: 'history'
        })
        // @ATTENTION: 执行到此处时需要重写matcher以覆盖之前注册的路由，因为addRoutes无法覆盖旧的重复路由，且官方不支持deleteRoutes方法
        this.$router.matcher = newRouter.matcher
        this.$router.addRoutes(defaultRootes.concat(filteredRoutes, redirectRoutes))
          
        // 4. 登录后路由跳转
        if (this.path.indexOf('/login') >= 0) {
            // 登录路由跳转到首页
            this.$router.replace('/')
        } else {
            let path = this.$route.fullPath
            // 需要有路由变化才能重新加载此前未注册的路由
            this.$router.replace('/loading')
            this.$router.replace(path)
        }
        // 4. 保存权限列表
        this.auths = response.data.auths
      }).catch(() => {
        // 未登录，跳转到登录页面
        this.goLogin()
      })
    }
}
```
##### 3. 登录过期处理
接口拦截器监测到token失效等鉴权失败等错误时，提示并跳转到登录页：
```
Vue.prototype
      .$confirm('登录失效，请重新登录', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        router.push('/login')
      }).catch(() => {
        router.push('/login')
      })
```


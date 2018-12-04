---
layout: post
title:  "d2-admin原版转换为飞冰版"
date:   2018-12-04 12:13:17 +0800
categories: d2-admin

---

# 一、相关资料与前期准备

- [d2-admin飞冰版](https://github.com/d2-projects/d2-admin-ice)
- [d2-admin原版](https://github.com/d2-projects/d2-admin/releases)

将下载d2-admin-ice项目从d2-admin-ice/vue-materials/scaffolds文件夹中拷贝出来，最为d2-admin飞冰版本，以下简称d2-admin-ice，原版d2-admin下载解压留作备用修改即可，原版d2-admin以下简称d2-admin

# 二、文件拷贝

- 将d2-admin-ice/.iceworks文件夹拷贝到d2-admin中
- 将d2-admin-ice/src/libs/util.ice.js文件夹拷贝到d2-admin/src/libs中


# 三、新建文件
- 1.在src下新建menuConfig.js

```
/* eslint-disable */

import util from '@/libs/util.ice'

// 菜单配置

// 侧栏菜单配置
// ice 会在新建页面的时候 push 数据
// ice 自动添加的菜单记录是以下格式：(不会有嵌套)
// {
//   name: 'Nav',
//   path: '/page',
//   icon: 'home',
// },

const asideMenuConfig = [

]

// 顶栏菜单配置
// ice 不会修改 headerMenuConfig
// 如果你需要功能开发之前就配置出菜单原型，可以只设置 name 字段
// D2Admin 会自动添加不重复 id 生成菜单，并在点击时提示这是一个临时菜单
const headerMenuConfig = [

]


// 参考设置顶栏菜单的方法 (vuex)
// $store.commit('d2adminMenuHeaderSet', menus)
// 设置侧边栏菜单的方法 (vuex)
// $store.commit('d2adminMenuAsideSet', menus)
// 你可以在任何地方使用上述方法修改顶栏和侧边栏菜单

// 导出顶栏菜单
export const iceMenuHeader = util.recursiveMenuConfig(headerMenuConfig)

// 导出侧边栏菜单
export const iceMenuAside = util.recursiveMenuConfig(asideMenuConfig)

```

- 2.在src下新建routerConfig.js

```
/* eslint-disable */

// 工具
import util from '@/libs/util.ice'
// 页面和布局
import HeaderAside from '@/layout/header-aside'

// 变量名 routerConfig 为 iceworks 检测关键字
// ice 会自动在这个变量下添加路由数据
// 请不要修改名称
// 备注 ice 自动添加的路由记录是以下格式
// {
//   path: '/page4',
//   layout: d2LayoutMain,
//   component: 4
// }

// 如果不指定 name 字段，会根据 path 生成 name = page-demo1
// 转换规则见 util.recursiveRouterConfig 中 path2name 方法
// meta 字段会和默认值使用 Object.assign 合并
// 如果不指定 meta.name 的话，name 字段会使用和上面路由 name 一样的取值逻辑
// 下面两个页面就是对比 你可以分别观察两个页面上显示的路由数据差异

const routerConfig = [

];


// 导出全部路由设置
// 这个数据会在 router.js 中被扁平处理

export default util.recursiveRouterConfig(routerConfig)
```

# 四、修改路由、菜单配置和util.ice.js

在d2-admin/src/router/routes.js文件中添加如下代码

```
import iceRouter from '@/routerConfig.js'

const frameIn = [
  // 在该变量（frameIn）的最后导入...iceRouter变量，在其中的元素可以视情况修改
  ...iceRouter
]
```

在d2-admin/src/menu/index.js文件中添加如下代码

```
import { iceMenuHeader, iceMenuAside } from '@/menuConfig.js'

export const menuAside = [
  ...iceMenuAside
]

export const menuHeader = [
  ...iceMenuHeader
]
```

改动完的完整util.ice.js内容如下

```
const util = {}
const meta = { requiresAuth: true }

/**
 * @description 路由配置扁平化
 * @param {Array} config 层级路由设置
 */
util.recursiveRouterConfig = function recursiveRouterConfig (config = []) {
  const routerMap = []
  /**
   * path -> name
   * @param {String} path path
   */
  function path2name (path = '') {
    return path.split('/').filter(e => e).join('-')
  }
  /**
   * recursive
   * @param {Array} con config
   */
  function recursive (con) {
    con.forEach((item) => {
      const route = item.layout ? {
        // -> 在主布局内的页面
        // 页面地址
        path: item.path,
        meta,
        redirect: { name: item.name || path2name(item.path) },
        // 使用的布局
        component: item.layout,
        // 子路由 一个里面只会有一个子路由 并且 path = ‘’
        children: [
          {
            // 这里留空 访问上面父级地址的时候会自动跳到这里
            path: 'index',
            // 如果路由没有设置 name 就用 path 处理成name
            name: item.name || path2name(item.path),
            // meta 设置和默认值合并
            meta: Object.assign({
              requiresAuth: true,
              title: path2name(item.path)
            }, item.meta),
            // 页面组件
            component: item.component
          }
        ]
      } : {
        // -> 不在主布局内的页面
        // 页面地址
        path: item.path,
        // 如果路由没有设置 name 就用 path 处理成name
        name: item.name || path2name(item.path),
        // meta 设置和默认值合并
        meta: Object.assign({
          requiresAuth: true,
          title: path2name(item.path)
        }, item.meta),
        // 页面组件
        component: item.component
      }
      if (Array.isArray(item.children)) {
        recursive(item.children)
      }
      routerMap.push(route)
    })
    return routerMap
  }
  return recursive(config)
}

/**
 * @description 转换菜单数据
 * @param {Array} arr menu config
 */
util.recursiveMenuConfig = function recursiveMenuConfig (arr) {
  const res = []
  /**
   * 转换每个菜单对象上的 name 为 title
   * @param {Object} obj menu
   */
  function convert (obj) {
    const {
      name, path, icon, children
    } = obj
    return {
      title: name,
      icon,
      path,
      ...children ? { children: children.map(convert) } : {}
    }
  }
  arr.forEach((menu) => {
    res.push(convert(menu))
  })
  return res
}

export default util
```
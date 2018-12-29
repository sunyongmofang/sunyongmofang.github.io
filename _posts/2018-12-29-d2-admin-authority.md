---
layout: post
title:  "d2-admin添加简单的权限控制"
date:   2018-12-29 14:36:58 +0800
categories: d2-admin

---

# 一、添加权限合并方法

在d2-admin项目中src/libs文件夹中添加util.auth.js文件，该js代码如下：


```
const auth = {}

/**
 * @description 判断 value 是否在 array 中
 * @param {String} value
 * @param {Array} array
 */
auth.includes = function (value, array) {
  for (let index in array) {
    if (array[index] === value) {
      return true
    }
  }
  return false
}

/**
 * @description 处理 menu ，将没有权限的菜单去除
 * @param {Array} menu cookie name
 * @param {Array} auths cookie value
 */
auth.merge = function (menu, auths) {
  let newMenu = []
  for (let i in menu) {
    let tmpMenu = Object.assign({}, menu[i])
    if (tmpMenu.auth && this.includes(tmpMenu.auth, auths)) {
      // if 条件为递归条件
      if (tmpMenu.children) {
        let tmpChildren = this.merge(tmpMenu.children, auths)
        tmpMenu.children = tmpChildren
      }
      // 基线操作
      newMenu.push(tmpMenu)
    }
  }
  return newMenu
}

export default auth

```

# 二、小修部分代码

- 1.在src/libs/util.js文件中添加并导出util.auth.js文件
- 2.在store/modules/d2admin/modules/account.js中的login方法中添加保存权限的动作，示例代码如下

```
// 在保存uuid和token的下方添加如下代码
util.cookies.set('auths', ['public', 'test'].join(','))
```

- 3.将src/menu/header.js中的菜单对象保存到header变量中（header请自行使用声明），添加如下代码

```
// 如下两行代码添加到src/menu/header.js文件到最上方
import util from '@/libs/util.js'

let auths = util.cookies.get('auths', '').split(',')

// 导出动作放在文件到最后，其中header变量为菜单对象（原始文件中导出对象为header）
export default util.auth.merge(header, auths)
```

- 4.在src/router/routes.js文件中修改如下代码

```
// 在meta变量声明处的上方添加如下两行代码
import util from '@/libs/util.js'

let auths = util.cookies.get('auths', '').split(',')


// 结尾处的两处导出动作改动如下（注意是“改动”）
export const frameInRoutes = util.auth.merge(frameIn, auths)

export default [
  ...util.auth.merge(frameIn, auths),
  ...frameOut,
  ...errorPage
]
```

# 三、具体使用方法

在菜单对象（在src/menu/header.js中）和frameIn路由对象（在src/router/routes.js中）的元素和每个children属性的每个元素中添加`auth`，`auth`属性值如果为public或test字符串（仅在本次测试中），则对应的菜单就可以正常展现，路由没有权限的则会抛出404
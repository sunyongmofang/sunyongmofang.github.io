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
 * @description 判断value是否在array中 复杂度为O(n)
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

/**
 * @description 获取所有权限
 * @param {Array} menu cookie name
 * @param {Array} auths cookie value
 */
auth.getAuths = function (routes) {
  let newAuths = []
  for (let i in routes) {
    let tmpNode = Object.assign({}, routes[i])
    if (tmpNode.meta && tmpNode.meta.name) {
      if (tmpNode.children) {
        let tmpChildren = this.getAuths(tmpNode.children)
        newAuths = [ ...newAuths, ...tmpChildren ]
      }
      newAuths.push(tmpNode.meta.name)
    }
  }
  let uniq = new Set(newAuths)
  return [ ...uniq ]
}

auth.auth2Name = function (auths, routes) {
  let names = []
  for (let i in routes) {
    let tmpNode = Object.assign({}, routes[i])
    if (tmpNode.meta && this.includes(tmpNode.meta.name, auths)) {
      if (tmpNode.children) {
        let tmpChildren = this.auth2Name(auths, tmpNode.children)
        names = [ ...names, ...tmpChildren ]
      }
      names.push(tmpNode.name)
    }
  }
  return names
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

- 3.在src/layout/header-aside/components/menu-header/index.vue和src/layout/header-aside/components/menu-side/index.vue导入util并修改如下代码

```
修改前：<template v-for="(menu, menuIndex) in header">
修改后：<template v-for="(menu, menuIndex) in headerShow">

//导入util
import util from '@/libs/util.js'

//在computed属性中添加如下方法
headerShow () {
  let auths = util.cookies.get('auths', '').split(',')
  return util.auth.merge(this.header, auths)
}
```

# 三、具体使用方法

在菜单对象（在src/menu/header.js中）的元素和每个children属性的每个元素中添加`auth`，`auth`属性值如果为public或test字符串（仅在本次测试中），则对应的菜单就可以正常展现，路由没有权限的则会抛出404，在路由中的meta属性中添加`auth`属性，`auth`属性值如果为public或test字符串即可正常展现访问该路由
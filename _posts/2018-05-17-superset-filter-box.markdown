---
layout: post
title:  "superset filter_box学习"
date:   2018-05-17 09:18:17 +0800
categories: test filter
---

# 一、如何让切片不受filter_box组件控制

## 1.filter_box组件介绍：

filter_box主要的功能为设置时间窗、过滤、筛选等功能，
在同一个看板中会无差别的设置切片的_from、_to、_filter等条件，导致看板中的所有切片全部刷新。

## 2.实现不受filter_box组件控制：

首先点击看板----》编辑相应的看板----》在json模版中的filter_immune_slices字段的值中添加相应的切片id

切片id可以在导航栏中点击“切片”，点击想要查看id的切片，在url地址栏中最后的数字为该切片id

# 二、看板中的json模板参数详解

```
{
    "filter_immune_slices": [], 
    "timed_refresh_immune_slices": [], 
    "expanded_slices": {
        "41": true
    }, 
    "filter_immune_slice_fields": {}, 
    "default_filters": "{\"41\":{\"sel_type\":[\"\u6240\u6709\"],\"__from\":\"1 days ago\"}}"
}
```

## 1.filter_immune_slices

参数为数组，在数组中直接填写相关切片的id，这样就可以达到该切片在此看板中不受filter_box控制

## 2.timed_refresh_immune_slices

参数形式与filter_immune_slices相同，主要功能为忽略对相关切片的定时刷新

## 3.expanded_slices

参数为map类型，key为字符串形式的切片id数，value为true或者false，主要功能为设置注释是否默认展现，true为展现，false为不展现

## 4.filter_immune_slice_fields

参数为map类型，key为字符串形式的切片id数，value为字符串数组，其中value中数组的值为屏蔽值，例子如下：

```
    "filter_immune_slice_fields": {
        "177": ["country_name", "__from", "__to"],
        "32": ["__from", "__to"]
    },
```

上面的例子中表示id为177的切片不受"country_name", "__from", "__to"字段影响

## 5.default_filters

设置filter_box的默认值

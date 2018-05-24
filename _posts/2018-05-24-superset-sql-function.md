---
layout: post
title:  "superset实现sql的存储过程调用"
date:   2018-05-24 10:45:17 +0800
categories: superset
---


# 一、修改相关文件

图片: https://images-cdn.shimo.im/JmEGEwI1HtIZwMt0/屏幕快照_2018_03_13_13.23.13.png

# 二、新增代码及其注释

1.在superset/assets/javascripts/explore/stores/controls.jsx中新增如下代码

{% highlight javascript %}
  sql: {
    type: 'TextAreaControl',
    label: t('Code'),
    description: t('Put your sql here. This feature is for developers who understand superset. If you are not sure, please do not fill in anything, it will be ignored.'),
    mapStateToProps: state => ({
      language: state.controls && state.controls.markup_type ? state.controls.markup_type.value : 'sql',
    }),
    default: '',
  },
{% endhighlight %}

新增代码添加到export const controls = {}对象中，该对象中到所有成员都以键值对（key:value）的形式存在。新增代码主要是添加sql语句的输入框，让superset管理员可以直接在页面编辑sql语句以完成更复杂的业务逻辑

2.superset/assets/javascripts/explore/stores/visTypes.js（一下简称visTypes.js文件）文件说明及其具体改动方法
在visTypes.js文件中export const sections = {}对象中的每个元素代表一个显示组件，而每个显示组件中的属性则代表了新建图表示时（点击explore chart按钮时）左侧显示的选项内容，如果想为某个显示组件添加sql编辑框，只需要在每个显示组件中的controlPanelSections属性中添加sql编辑框即可，具体实例如下：

{% highlight javascript %}
      {
        label: t('Code'),
        controlSetRows: [
          ['sql'],
        ],
      },
{% endhighlight %}

因为controlPanelSections属性的值为数组，只需要在该数组中的任意位置插入如上代码即可在新建（编辑）页面看到sql编辑框。

3.在superset/connectors/sqla/models.py文件中的SqlaTable类中添加如下方法：

{% highlight python %}
    def get_query_str(self, query_obj):
        engine = self.database.get_sqla_engine()
        if query_obj.has_key('form_data') and query_obj['form_data'].has_key('sql') and query_obj['form_data']['sql']:
            dttm_to_str = [x for x in self.columns if x.is_dttm][0].dttm_sql_literal
            query_obj['from_dttm'], query_obj['to_dttm'] = dttm_to_str(query_obj['from_dttm']), dttm_to_str(query_obj['to_dttm'])
            sql = query_obj['form_data']['sql']
            for k, v in query_obj.iteritems():
                if isinstance(v, basestring):
                    sql = sql.replace(''.join(['{', k, '}']), v)
            for v in query_obj['form_data']['filters']:
                tmp_key, tmp_value = v['col'], v['val'][0]
                sql = sql.replace(''.join(['{', tmp_key, '}']), tmp_value)
        else:
            qry = self.get_sqla_query(**query_obj)
            sql = six.text_type(
                qry.compile(
                    engine,
                    compile_kwargs={'literal_binds': True},
                ),
            )
        logging.info(sql)
        sql = sqlparse.format(sql, reindent=True)
        return sql
{% endhighlight %}

该方法会判断如果前端sql代码输入框有内容则直接执行sql语句，并返回相应的结果

4.编译修改结果

在incubator-superset/superset/assets文件夹下运行js_build.sh脚本编译前端代码，python代码在incubator-superset文件夹中使用如下命令进行编译安装

{% highlight shell %}
python setup.py build
python setup.py install
{% endhighlight %}

# 三、最终效果以及使用方法

1.显示效果

图片: https://images-cdn.shimo.im/AImFE2TxJMIBV17n/屏幕快照_2018_03_13_13.41.18.png在visTypes.js文件中添加了Code标签的选项的组件会有code选项，在其中可以直接编写sql语句

2.使用方法

在编写sql语句时支持参数的传递，参数以{}进行标识，执行时会自动替换为相应的动态参数，常用的参数有from_dttm、to_dttm，from_dttm、to_dttm分别代表开始时间和结束时间，时间参数不用加引号（'），其余的参数可以根据filter_box中的选项进行动态的修改。


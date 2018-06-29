---
layout: post
title:  "python链接hive"
date:   2018-06-05 10:20:17 +0800
categories: python
---

# 一、代码示例

{% highlight python %}
from impala.dbapi import connect


def execute_sql(sql):
    results = None
    with connect(host='110.18.18.121', port=10015, database='aws', auth_mechanism='PLAIN') as conn:
        with conn.cursor() as cursor:
            cursor.execute(sql)
            results = cursor.fetchall()
    return results
{% endhighlight %}
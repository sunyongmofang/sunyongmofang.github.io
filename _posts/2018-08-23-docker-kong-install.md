---
layout: post
title:  "docker环境下部署kong"
date:   2018-08-23 11:47:17 +0800
categories: docker

---

# 一、创建docker网络

{% highlight shell %}
$ docker network create kong-net
{% endhighlight %}

docker网络名为kong-net，此名字可以使用任意字符串代替，不过要注意修改后续命令中使用的网络名

# 二、安装docker数据库（PostgreSQL）

{% highlight shell %}
$ docker run -d --name kong-database \
              --network=kong-net \
              -p 5432:5432 \
              -e "POSTGRES_USER=kong" \
              -e "POSTGRES_DB=kong" \
              postgres:9.6
{% endhighlight %}

安装postgres时要注意端口占用

# 三、kong初始化数据库

{% highlight shell %}
$ docker run --rm \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong:latest kong migrations up
{% endhighlight %}


# 四、启动kong

{% highlight shell %}
$ docker run -d --name kong \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    -p 8000:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    kong:latest
{% endhighlight %}

# 五、安装kong-dashboard

{% highlight shell %}
$ docker run --network=kong-net --name kong-dashboard -d -p 8084:8080 pgbi/kong-dashboard:v2
{% endhighlight %}

访问8084端口后填入kong的ip地址加8001端口即可完成设置


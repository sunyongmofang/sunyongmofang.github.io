---
layout: post
title:  "kong的简单使用方法"
date:   2018-09-07 18:10:17 +0800
categories: docker

---

# 一、Kong简介

Kong是一个云原生，快速，可扩展和分布式微服务抽象层（也称为API网关，API中间件或在某些情况下为Service Mesh）。作为2015年的开源项目，其核心价值在于高性能和可扩展性。


# 二、创建测试接口

{% highlight shell %}
$ docker run -d --name prest4test \
              -p 6000:6000 \
              -v /yourlocalpath/tmp:\tmp \
              -v /yourlocalpath/prest.toml:prest.toml \
              prest/prest
{% endhighlight %}

以上命令为安装prest的docker命令，注意要提前安装PostgreSQL并注意prest.toml文件中PostgreSQL的ip配置

prest详细信息可前往[prest官网](https://postgres.rest/)查看详细信息

# 三、Kong详细使用方法

## 1.测试prest

{% highlight shell %}
$ curl -X GET http://localhost:6000/databases
{% endhighlight %}

以上命令返回PostgreSQL中所有的databases即为正常，切记不要尝试在浏览器中打开上面的链接:)

## 2.将prest接口（api）对接到kong

kong出口端口（默认HTTP 8000，HTTPS 8443）

kong管理端口（默认8001）

### 1.注册新api

{% highlight shell %}
$ curl -i -X POST http://localhost:8001/apis/ \
    -d "name=testapi" \
    -d "request_path=/test" \
    -d "upstream_url=http://localhost:6000/" \
    -d "strip_request_path=true"
{% endhighlight %}

如上命令即可注册一个名为testapi的api到/test路径下，kong出口端口加上/test就可以转发到http://localhost:6000/

### 2.测试kong中新添加的api

{% highlight shell %}
$ curl -i -X GET http://localhost:8000/test/databases
{% endhighlight %}

如上命令中http://localhost:8000/test会被解析到http://localhost:6000，/databases路径会被发送到http://localhost:6000接口中

上条命了如果返回结果和`1.测试prest`中的结果相同即可

### 3.给api加上基本的权限验证

{% highlight shell %}
$ curl -X POST http://localhost:8001/apis/testapi/plugins -d "name=key-auth"
{% endhighlight %}

testapi加上key-auth插件如果没有头部验证就无法正常访问了，如果想要正常访问就要添加身份认证

### 4.创建一个新的Consumer

{% highlight shell %}
$ curl -X POST http://localhost:8001/consumers/ -d "username=testuser"
{% endhighlight %}

然后，我们在key-auth插件中为它创建一个key

{% highlight shell %}
$ curl -X POST http://localhost:8001/consumers/testuser/key-auth -d "key=testkey"
{% endhighlight %}

这次加上apikey首部进行验证,成功调用

{% highlight shell %}
$ http://localhost:8001/test/databases --header "apikey: testkey"
{% endhighlight %}

其上命令即可以正常展现数据，这样就可以实现通过kong实现api权限的控制:)



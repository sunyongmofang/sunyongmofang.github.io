---
layout: post
title:  "gfw调研与安装"
date:   2018-04-11 10:12:17 +0800
categories: gfw

---

# 一、相关资料

- [shadowsocks（必须）](https://github.com/shadowsocks/shadowsocks-libev)
- [google bbr（可选）](https://github.com/iMeiji/shadowsocks_install/wiki/%E5%BC%80%E5%90%AFTCP-BBR%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E7%AE%97%E6%B3%95)
- [pac文件语法（可选）](http://www.devnotes.in/2014/11/08/auto-proxy-settings-with-PAC.html)


# 二、centos普通用户下安装（需要sudo权限）

{% highlight shell %}
$ yum install epel-release -y
$ yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto c-ares-devel libev-devel libsodium-devel mbedtls-devel git vim -y
$ git clone https://github.com/shadowsocks/shadowsocks-libev.git
$ cd shadowsocks-libev
$ git submodule update --init --recursive
$ ./autogen.sh && ./configure && make
$ sudo make install
{% endhighlight %}

至此ss相关命令全部安装完毕

# 三、ss服务端配置

{% highlight shell %}
$ ss-server \
    -s your_ip \
    -p server_port \
    -l client_port \
    -k xxxxx \
    -m method \
    -f /your_path/ss-server.pid \
    -t number \
    -u
{% endhighlight %}

参数注解：
-s 指定对外本机服务地址，通常为0.0.0.0(0.0.0.0指本机完全对外暴露)
-p 绑定服务端的端口号
-l 指定客户端的端口号
-k 设置ss服务密码
-m 指定加密方式，支持加密方式如下：
```
aes-128-gcm, aes-192-gcm, aes-256-gcm,
aes-128-cfb, aes-192-cfb, aes-256-cfb,
aes-128-ctr, aes-192-ctr, aes-256-ctr,
camellia-128-cfb, camellia-192-cfb,
camellia-256-cfb, bf-cfb,
chacha20-ietf-poly1305,
xchacha20-ietf-poly1305,
salsa20, chacha20 , rc4-md5, chacha20-ietf
```
-f 保存进程号的文件地址，使用此参数ss会以守护进程的方式启动
-t 设置超时时间
-u 支持udp（如果玩海外游戏就加上，-u后面不用加任何参数）

# 四、ss客户端下载（GUI）

- [windows](https://github.com/shadowsocks/shadowsocks-windows/releases)
- [macos](https://github.com/shadowsocks/ShadowsocksX-NG/releases)

---

google bbr（可选）

通过更换内核（>4.9）使用bbr，即可省去双边加速到累赘，又有不俗到加速效果，并且基本不用配置，操作极其简单
[设置方法](https://github.com/iMeiji/shadowsocks_install/wiki/%E5%BC%80%E5%90%AFTCP-BBR%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E7%AE%97%E6%B3%95)

---
mac代理设置命令（可选）：networksetup

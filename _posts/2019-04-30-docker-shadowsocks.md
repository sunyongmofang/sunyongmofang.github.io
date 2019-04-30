---
layout: post
title:  "docker搭建shadowsocks"
date:   2019-04-30 19:38:17 +0800
categories: gfw
---

# 一、新建服务器

在google云上新建个服务器，服务器系统为`Container-Optimized OS`，等待服务器启动后即可使用ssh等其他方式登录即可

# 二、部署shadowsocks容器

```
docker run \
    -e PASSWORD=password \
    -e METHOD=method \
    --name shadowsocks \
    -p 8388:8388 \
    -p 8388:8388/udp \
    -d --restart always \
    shadowsocks/shadowsocks-libev
```

PASSWORD值为shadowsocks密码
METHOD值为如下加密方式

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

-p参数`:`左边的端口号可以替换为其他端口号，左边端口号会映射到主机上作为shadowsocks对外端口
---
layout: post
title:  "gfw调研"
date:   2018-04-11 10:12:17 +0800
---


- [shadowsocks](https://github.com/shadowsocks/shadowsocks-libev)
- [kcptun](https://github.com/xtaci/kcptun/issues/137)
- [kcptun配置方法](https://github.com/xtaci/kcptun/issues/137)
- [kcptun配置组合](https://github.com/xtaci/kcptun/issues/251)
- [kcptun参数解析](https://blog.kuoruan.com/102.html)


---
shadowsocks工作流程
```
graph TD
A(浏览器)
B(ss-local)
C(ss-server)
D(server)
A-->|浏览器进入等待状态|B
B-->|ss-local进入等待状态|C
C-->|ss-server收集数据|D
D-->|数据发送到ss-server|C
C-->|数据发送到ss-local|B
B-->|数据发送到浏览器|A
```

---
```
graph TD
A(kcp-server)
B(kcp-local)
C(ss-server)
D(ss-local)
E(server)
F(浏览器)
F-->|不支持的请求发送到ss|D
D-->|本地ss加密后|B
B-->|使用特殊到发包技术降低延迟|A
A-->|ss解密数据|C
C-->|server获取网络数据|E
```

---

google bbr

通过更换内核（>4.9）使用bbr，即可省去双边加速到累赘，又有不俗到加速效果，并且基本不用配置，操作极其简单
[设置方法](https://github.com/iMeiji/shadowsocks_install/wiki/%E5%BC%80%E5%90%AFTCP-BBR%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E7%AE%97%E6%B3%95)

---
mac代理设置方法

networksetup

http://www.devnotes.in/2014/11/08/auto-proxy-settings-with-PAC.html
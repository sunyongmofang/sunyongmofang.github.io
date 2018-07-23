---
layout: post
title:  "frp安装记录"
date:   2018-07-23 11:03:56 +0800
categories: frp
---

# 一、下载应用

[frp下载地址][frp_download]

# 二、服务端配置

{% highlight shell %}
$ cat frps.ini
[common]
bind_port = 7000

$ ./frps -c frps.ini
{% endhighlight %}


# 三、客户端配置

{% highlight shell %}
$ cat frpc.ini
[common]
server_addr = XX.XX.XX.XX #此处应该为服务端的ip或域名
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

$ ./frpc -c frpc.ini
{% endhighlight %}

# 四、验证方式及其注意事项

{% highlight shell %}
$ ssh -p 6000 user@XX.XX.XX.XX
{% endhighlight %}

如上命令中`user`更换为客户端用户名，`XX.XX.XX.XX`更换为服务端的ip或域名，如果可以成功登录客户端则配置成功（可能需要输入客户端登录密码）

注意事项：
- 服务端和客户端必须要连接网络:)
- 服务端必须有外网ip或域名

[frp_download]: https://github.com/fatedier/frp/releases
---
layout: post
title:  "vim常用操作"
date:   2019-12-28 13:55:17 +0800
categories: gfw 
---

# 零、前期准备

域名、ssl证书、域名绑定vps的ip、ssl证书上传到服务器

# 一、卸载阿里云相关组件

执行如下命令卸载阿里云相关监控程序

```
wget http://update.aegis.aliyun.com/download/uninstall.sh && chmod +x uninstall.sh && ./uninstall.sh
wget http://update.aegis.aliyun.com/download/quartz_uninstall.sh && chmod +x quartz_uninstall.sh && ./quartz_uninstall.sh
/usr/local/cloudmonitor/wrapper/bin/cloudmonitor.sh stop
/usr/local/cloudmonitor/wrapper/bin/cloudmonitor.sh remove
rm -rf /usr/local/cloudmonitor
rm -rf /usr/local/aegis
rm /usr/sbin/aliyun-service
rm /lib/systemd/system/aliyun.service
rm quartz_uninstall.sh uninstall.sh
apt-get -y update
apt-get -y upgrade
```

# 二、安装v2ray

执行如下命令安装v2ray并设置开机自启

```
bash <(curl -L -s https://install.direct/go.sh)
systemctl enable v2ray
```

`/etc/v2ray/config.json`文件内容如下，id和path请自行替换

```
{
  "inbounds": [
    {
      "port": 10000,
      "listen": "127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/something/"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

# 三、安装nginx

执行如下命令安装nginx

```
apt-get -y install nginx
```

将ssl证书放入/etc/nginx/conf.crt目录下并重命名为api.key和api.pem，如果没有/etc/nginx/conf.crt目录请自行创建，将/etc/nginx/nginx.conf文件修改为如下内容

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 10240;
}

http {

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_certificate_key /etc/nginx/conf.crt/api.key;
	ssl_certificate /etc/nginx/conf.crt/api.pem;

	gzip on;
	gzip_disable "msie6";

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

将/etc/nginx/sites-available/default文件修改为如下内容，/something/请自行修改（应修改为和v2ray配置文件`/something/`相同自负）

```
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	listen 443 ssl default_server;
	listen [::]:443 ssl default_server;

	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	location /something/ {
		if ($http_upgrade != "websocket") {
			return 404;
		}
		proxy_redirect off;
		proxy_pass http://127.0.0.1:10000;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}

}
```

# 四、安装bbr_power

执行如下命令安装BBR_POWER

```
wget --no-check-certificate -qO 'BBR_POWERED.sh' 'https://moeclub.org/attachment/LinuxShell/BBR_POWERED.sh' && chmod 755 BBR_POWERED.sh && ./BBR_POWERED.sh
wget --no-check-certificate -qO 'BBR.sh' 'https://moeclub.org/attachment/LinuxShell/BBR.sh' && chmod 755 BBR.sh && ./BBR.sh -f
./BBR_POWERED.sh
```

# 五、客户端配置

如下图中所示，Address中填写域名、User ID中应与v2ray配置中的id一致

![主页面配置](http://git.aiou.xyz/images/v2ray_client1.png)

点击transport settings...，WebSocket配置如下，/something/应与v2ray配置中的path一致

![WebSocket配置](http://git.aiou.xyz/images/v2ray_client2.png)

TLS配置如下，TLS ServerName中填写用户名

![TLS配置](http://git.aiou.xyz/images/v2ray_client3.png)
![优化配置](http://git.aiou.xyz/images/v2ray_client4.png)

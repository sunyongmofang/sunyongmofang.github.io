---
layout: post
title:  "树莓派初次安装"
date:   2018-11-30 14:21:17 +0800
categories: raspberry

---

# 一、相关资料与前期准备

- [树莓派官网](https://www.raspberrypi.org/)
- [参考文档](https://blog.csdn.net/kxwinxp/article/details/78370913)
- [树莓派官方镜像下载地址（必备，需要下载相应镜像）](https://www.raspberrypi.org/downloads/raspbian/)
- [数据烧写工具（必备，需要下载相应镜像）](https://www.balena.io/etcher/)

# 二、镜像烧写到sd卡中

使用`etcher`软件将`官方镜像`烧写到sd卡中，操作比较简单，不过多赘述，如有问题请参阅参考文档

# 三、开启ssh并设置开机Wi-Fi连接

镜像烧写完毕后建议重新插拔内存卡，以便电脑重新识别内存卡。

此时如果一切正常，电脑将会把内存卡识别为boot，windows中直接在`我的电脑`中找到内存卡并双击进入操作，mac系统中可以使用命令行`cd /Volumes/boot`来进入内存卡进行操作，至此就进入了内存卡中，此操作以下称为`boot分区中`。

在boot分区中新建wpa_supplicant.conf文件，在wpa_supplicant.conf文件中保存如下内容

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="XXXXXXX"
    psk="XXXXXXXX"
    priority=1
}
```

ssid的值为Wi-Fi名
psk的值为Wi-Fi密码


# 四、获取树莓派ip的方法

- 1.在路由中查看新增ip
- 2.在第三方或其他渠道获取树莓派ip，比如企业多数都有ip管理页面

使用终端命令即可连接树莓派，默认密码为raspberry

```
ssh pi@raspberry_ip
```

请将上面的raspberry_ip替换为树莓派只是ip

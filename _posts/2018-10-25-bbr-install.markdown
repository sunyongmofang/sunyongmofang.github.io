---
layout: post
title:  "google bbr配置方法"
date:   2018-03-29 12:34:17 +0800
categories: gfw
---

# 一、安装内核

{% highlight shell %}
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
# yum --enablerepo=elrepo-kernel install kernel-ml -y
# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
# grub2-set-default 0
# shutdown -r now
{% endhighlight %}

注意：手动安装高版本内核后切勿在vps运营商控制面板里选择重启，否则有可能运营商会使用原有内核启动，请使用命令在终端重启

# 二、更新内核并启动bbr

## 1.更新内核
{% highlight shell %}
# yum --enablerepo=elrepo-kernel update -y
# shutdown -r now
{% endhighlight %}

## 2.启用bbr

{% highlight shell %}
# modprobe tcp_bbr
# echo "tcp_bbr" | sudo tee --append /etc/modules-load.d/modules.conf
# echo "net.core.default_qdisc=fq" | sudo tee --append /etc/sysctl.conf
# echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee --append /etc/sysctl.conf
# sysctl -p
{% endhighlight %}

## 3.验证bbr是否启动

如下命令均有bbr打印则代表bbr启用成功

{% highlight shell %}
# sysctl net.ipv4.tcp_available_congestion_control
# sysctl net.ipv4.tcp_congestion_control
# lsmod | grep bbr
{% endhighlight %}
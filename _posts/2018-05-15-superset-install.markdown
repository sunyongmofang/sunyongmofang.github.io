---
layout: post
title:  "superset安装部署文档"
date:   2018-05-15 09:20:17 +0800
categories: superset
---


# 一、使用python virtualenv环境

## 1.建立并安装virtualenv环境
```
pip install virtualenv
virtualenv venv
. ./venv/bin/activate
```

使用pip安装virtualenv时可能需要root权限，第二行为新建虚拟环境，最后一行为运行虚拟环境，结束虚拟环境使用deactivate命令即可

## 2.在mac中安装系统依赖

```
brew install pkg-config libffi openssl
env LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include" pip install cryptography==1.7.2
```

如上命令即可在mac中安装相应依赖

`另附`：如果是线上centos环境，可执行如下命令

```
sudo yum upgrade python-setuptools
sudo yum install gcc gcc-c++ libffi-devel python-devel python-pip python-wheel openssl-devel libsasl2-devel openldap-devel
```

如上命令即可完成centos的系统依赖

# 二、pg数据库安装

## 1.安装以及初始化

```
brew install postgresql -v
initdb /usr/local/var/postgres -E utf8
```

## 2.启动pg数据库

```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```

## 3.停止数据库

```
pg_ctl -D /usr/local/var/postgres stop -s -m fast
```

## 4.创建用户

```
createuser username -P
```

## 5.登陆数据库

```
createdb dbname -O username -E UTF8 -e
```

数据库的安装较为简单，值得注意的是superset对高版本的pg数据库支持更友好，在Linux中尽量不要使用yum安装默认版本

# 三、superset编译与安装

## 1.从GitHub中检出superset

```
git clone https://github.com/apache/incubator-superset.git
```

## 2.superset前端编译

使用源代码暗转时必须编译前端代码，$SUPERSET_HOME变量代表superset主目录（下同），通常为incubator-superset文件夹

```
cd $SUPERSET_HOME/superset/assets
./js_build.sh
```

## 3.superset编译并安装

```
cd $SUPERSET_HOME
python setup.py install
```

## 4.superset初始化并启动运行

```
fabmanager create-admin --app superset
superset db upgrade
superset load_examples
superset init
superset runserver 
```





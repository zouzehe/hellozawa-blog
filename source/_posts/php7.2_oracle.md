---
title ubuntu 18.04 安装php7.2 oci8 pdo_oci
date: 2019-04-29 05:37:46
tags: 
    - php 
    - oracle
---

## 前言

最近在一个项目中，需要把数据存储到oracle数据库中，听说oracle client安装比较麻烦，谨以此文记录一下安装过程

## 安装php7.2

我是源代码安装的

```shell
apt-get install libxml2-dev libcurl4-gnutls-dev libbz2-dev libjpeg-dev libpng-dev libxpm-dev libfreetype6-dev libt1-dev libmcrypt-dev libmysql++-dev libxslt1-dev

wget https://www.php.net/distributions/php-7.2.17.tar.gz
tar -xzvf php-7.2.17.tar.gz
cd php-7.2.17
./configure --prefix=/data/server/php-7.2 --with-config-file-path=/data/server/php-7.2/etc --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-mbstring --enable-ftp --with-gd --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --disable-fileinfo --enable-opcache --enable-maintainer-zts --with-config-file-scan-dir=/data/server/php-7.2/etc/conf.d --with-config-file-path=/data/server/php-7.2/etc/ --with-kerberos
make
make install
```
## 安装Oracle Instant Client 和 SDK
1. 首先去oracle官网 https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html ，下载需要登录验证的，没有账户的可以先注册。
- instantclient-basic-linux.x64-19.3.0.0.0dbru.zip 
- instantclient-sdk-linux.x64-19.3.0.0.0dbru.zip 

2. mkdir /opt/oracle
3. cd /opt/oracle
   unzip instantclient-basic-linux.x64-19.3.0.0.0dbru.zip
   unzip instantclient-sdk-linux.x64-19.3.0.0.0dbru.zip
4. 添加ldconfig
```
   echo /opt/oracle/instantclient_19_3 > /etc/ld.so.conf.d/oracle-instantclient.conf
```
5. 执行 ldconfig 生效
## 编译安装oci8
1. apt-get install build-essential libaio1
2. wget http://pecl.php.net/get/oci8-2.2.0.tgz
解压进入目录
./configure --with-php-config=/data/server/php-7.2/bin/php-config --with-oci8=instantclient,--with-oci8=instantclient,/opt/oracle/instantclient_19_3
make
3. 中间可能会报错
```
running: make
/bin/sh /var/tmp/pear-build-rootMIYqiX/oci8-2.0.8/libtool --mode=compile cc  -I. -I/var/tmp/oci8 -DPHP_ATOM_INC -I/var/tmp/pear-build-rootMIYqiX/oci8-2.0.8/include -I/var/tmp/pear-build-rootMIYqiX/oci8-2.0.8/main -I/var/tmp/oci8 -I/usr/include/php -I/usr/include/php/main -I/usr/include/php/TSRM -I/usr/include/php/Zend -I/usr/include/php/ext -I/usr/include/php/ext/date/lib -I/usr/include/oracle/11.2/client64  -DHAVE_CONFIG_H  -g -O2   -c /var/tmp/oci8/oci8.c -o oci8.lo
libtool: compile:  cc -I. -I/var/tmp/oci8 -DPHP_ATOM_INC -I/var/tmp/pear-build-rootMIYqiX/oci8-2.0.8/include -I/var/tmp/pear-build-rootMIYqiX/oci8-2.0.8/main -I/var/tmp/oci8 -I/usr/include/php -I/usr/include/php/main -I/usr/include/php/TSRM -I/usr/include/php/Zend -I/usr/include/php/ext -I/usr/include/php/ext/date/lib -I/usr/include/oracle/11.2/client64 -DHAVE_CONFIG_H -g -O2 -c /var/tmp/oci8/oci8.c  -fPIC -DPIC -o .libs/oci8.o
In file included from /var/tmp/oci8/oci8.c:46:
/var/tmp/oci8/php_oci8_int.h:48:29: error: oci8_dtrace_gen.h: No such file or directory
make: *** [oci8.lo] Error 1
ERROR: `make' failed
```
4. 解决办法，安装dtrace
apt-get install systemtap-sdt-dev
然后基本上没问题了
5. 重新make && make install
6. 加入php.ini 
echo "extension =oci8.so" >> /data/server/7.2/etc/php.ini
## 安装php ext pdo_oci
1. 直接接入php源码包目录
```
cd /data/tmp/php-7.2.17/ext/pdo_oci
phpize
./configure --with-php-config=/data/server/php-7.2/bin/php-config  --with-pdo-oci=instantclient,/opt/oracle/instantclient_19_3
make
make install
vi /data/server/7.2/etc/php.ini
extension=pdo_oci 把注释去掉
```
## 至此安装完毕
php -m
 oci8
 PDO_OCI

 
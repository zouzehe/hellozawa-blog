---
title: postfix mail
date: 2018-05-08 04:44:47
tags:
    - postfix
    - email
    - linux
---

CentOS安装postfix、mailx利用QQ企业邮箱的smtp服务器通过命令行发送邮件
个人VPS上搭建邮件服务器比较繁琐，还容易被当做垃圾邮件服务器列入黑名单。
简单的方法就是可以配置SMTP通过Gmail或者QQ邮箱发送邮件。
本站使用的服务器是CentOS 6.5的，在上面配置了postfix+mailx，针对QQ邮箱SMTP非加密认证方式很简单，网上随便搜一下都有大堆教程。由于服务器放在国外，不能了解机房的真正环境，对非加密方式的信息传输很容易被其他服务器sniff到密码和邮件内容，当然在国内也是，配置SMTP最好是使用SSL加密（Ps:本人曾经遇到sniff到FTP密码的情况，因此强烈建议服务器上涉及密码的都用SSL传输）。

【postfix+mailx+QQ邮箱】CentOS配置方法：

### 1、不管有没有先卸载掉sendmail吧，并安装postfix+mailx
```shell
#卸载sendmail
service sendmail stop
chkconfig sendmail off
yum remove sendmail
#安装postfix mailx和sasl需要的包
yum install postfix mailx cyrus-sasl-plain
```

### 2、配置QQ邮箱SMTP账号信息
先看下QQ邮箱帮助，SSL端口可用465或者587
如何设置POP3/SMTP的SSL加密方式？
使用SSL的通用配置如下：
接收邮件服务器：pop.qq.com，使用SSL，端口号995
发送邮件服务器：smtp.qq.com，使用SSL，端口号465或587

接着创建密码配置文件 
```shell
/etc/postfix/sasl_passwd  账号密码用分号:隔开
echo "[smtp.qq.com]:587    xxxxxxxx@qq.com:xxxxxxxxx" > /etc/postfix/sasl_passwd
```
密码是明文存储的，通过postmap创建hash加密文件sasl_passwd.db
最后测试没问题就可以删除sasl_passwd
```shell
postmap hash:/etc/postfix/sasl_passwd
```

### 3、创建ca证书，配置main.cf
```shell
#进入certs目录，先修改创建的证书时间，默认有效期只有一年，改成10年先
cd /etc/ssl/certs/
vi Makefile
找到里面所有的 -days 365 改为 -days 3650 也可以随便定个时间
保存退出后执行make
#创建证书
make server.pem
```
然后出现填写地区信息
```shell
[root@localhost certs]# make server.pem
umask 77 ; \
PEM1=`/bin/mktemp /tmp/openssl.XXXXXX` ; \
PEM2=`/bin/mktemp /tmp/openssl.XXXXXX` ; \
/usr/bin/openssl req -utf8 -newkey rsa:2048 -keyout $PEM1 -nodes -x509 -days 3650 -out $PEM2 -set_serial 0 ; \
cat $PEM1 >  server.pem ; \
echo ""    >> server.pem ; \
cat $PEM2 >> server.pem ; \
rm -f $PEM1 $PEM2
Generating a 2048 bit RSA private key
............+++
..........................+++
writing new private key to '/tmp/openssl.hZa44C'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:GUANGDONG
Locality Name (eg, city) [Default City]:SHENZHEN
Organization Name (eg, company) [Default Company Ltd]:TENCENT
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:
```
创建完成后会生成server.pem
#移动到/etc/postfix/目录
```shell
mv server.pem /etc/postfix/
```
重要：配置 generic 替换系统发信人，不然当你以root用户执行mailx的时候，发信人将是root@qq.com
这当然是不可能被腾讯允许的，会报错误(mail from address must be same as authorization user) 如下：
```
Sep 21 07:07:01 WEBSERVER postfix/qmgr[9086]: 43BD14953: from=<root@qq.com>, size=1363, nrcpt=1 (queue active)
Sep 21 07:07:02 WEBSERVER postfix/smtp[9102]: 43BD14953: to=<test@domain.com>, relay=smtp.qq.com[163.177.65.157]:587, delay=1, delays=0.02/0.03/0.91/0.08, dsn=5.0.0, status=bounced (host smtp.qq.com[163.177.65.157] said: 501 mail from address must be same as authorization user (in reply to MAIL FROM command))
```
需要在/etc/postfix/generic下添加账号映射，其他的服务器账号同理都要添加上。
```shell
echo 'root@qq.com    xxxxxx@qq.com' >>/etc/postfix/generic
#并生成generic.db文件
postmap /etc/postfix/generic
```
然后配置main.cf，备份原文件，然后清空main.cf，替换如下：
```shell
mydomain = qq.com
myorigin = $mydomain
myhostname = $mydomain
mydestination = $mydomain
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
relayhost = [smtp.qq.com]:587
smtp_use_tls = yes
smtp_tls_CAfile = /etc/postfix/server.pem
smtp_generic_maps = hash:/etc/postfix/generic
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
```

### 4、先再开一个终端，实时查看maillog发送日志
```shell
tail -f /var/log/maillog
```
最后重启postfix，发送测试邮件
```shell
service postfix restart
#测试发送，最好先测试发送到其他域的邮件，例如163.com
echo 'This is a test mail' | mail -s 'This is a test mail' xxxxx@163.com
```
如果按照这个步骤来就可以正常发送成功~失败的话看看错误日志，用Goolge搜，国内信息太少~
最后的最后，测试没问题就把明文保存的邮箱密码文件删掉吧！
```shell
rm -f /etc/postfix/sasl_passwd
```
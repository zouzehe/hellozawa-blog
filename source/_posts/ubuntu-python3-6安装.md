---
title: ubuntu python3.6安装
date: 2018-07-25 03:08:05
tags: Linux
---
# 1 安装依赖包
```shell
apt-get update && apt-get -y upgrade
apt-get -y install wget libkrb5-dev libsqlite3-dev gcc make automake libssl-dev zlib1g-dev libmysqlclient-dev libffi-dev git

# 修改字符集，否则可能报 input/output error的问题，因为日志里打印了中文
apt-get install language-pack-zh-hans
echo 'LANG="zh_CN.UTF-8"' > /etc/default/locale

```
# 2 编译安装
```shell
wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tar.xz
tar xvf Python-3.6.1.tar.xz  && cd Python-3.6.1
./configure --prefix=/path/to/python3.6
make
make install
```
# 3 建立 Python 虚拟环境
为了不扰乱原来的环境我们来使用 Python 虚拟环境
```shell
cd /opt
python3.6 -m venv py3
source /opt/py3/bin/activate
```

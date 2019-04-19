---
title: 单实例控制shell
date: 2018-05-08 05:28:19
tags: shell
---

工作中经常需要对脚本进行实例控制，以下提高一种思路

```shell
#!/bin/sh
auto_run_file=`echo $PWD $*|sed 's/ \|\//_/g'`
pidfile=/tmp/only_one_php_"$auto_run_file".pid
wk=`date +%w`
 
if [ -e $pidfile ]
then
   pid=`cat $pidfile`
   pid_ps=`ps -ef|grep $pid|awk '{ print $2 }'`
   for apid in $pid_ps
   do
        if [ $apid -eq $pid ]
        then
                exit
        fi
   done
fi
echo $$ >$pidfile
if [ -z $auto_run_file ]
then
        echo "参数为空"
                exit
fi
if [ ! -d /tmp/$auto_run_file ]
then
        /bin/mkdir -p /tmp/$auto_run_file
fi
echo -n "#############start " >/tmp/$auto_run_file/"$auto_run_file"_$wk.log
date +%Y%m%d/%T >>/tmp/$auto_run_file/"$auto_run_file"_$wk.log
/usr/local/bin/php $* >>/tmp/$auto_run_file/"$auto_run_file"_$wk.log
echo -n "#############end " >>/tmp/$auto_run_file/"$auto_run_file"_$wk.log
date +%Y%m%d/%T >>/tmp/$auto_run_file/"$auto_run_file"_$wk.log
rm -f $pidfile

```
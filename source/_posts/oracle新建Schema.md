---
title: oracle新建Schema命令
date: 2019-07-01 05:37:46
tags: oracle
---

```sql
-- 查看当前已有的用户
SELECT Username FROM dba_users;

-- 创建临时
CREATE USER gzmpc IDENTIFIED BY PASSWORD;

-- 授权
GRANT CREATE SESSION TO gzmpc;

CREATE TABLESPACE gzmoc_wk DATAFILE 'gzmoc_wk.dat' SIZE 10M AUTOEXTEND ON;

CREATE TEMPORARY TABLESPACE gzmoc_wk_tablespace_temp TEMPFILE 'gzmoc_tabspace_temp.dat' SIZE 5M AUTOEXTEND ON;

DROP USER gzmpc;

-- 开始创建数据库
CREATE USER gzmpc
IDENTIFIED BY PASSWORD
DEFAULT TABLESPACE gzmoc_wk
TEMPORARY TABLESPACE gzmoc_wk_tablespace_temp;

-- 授权
grant create session to gzmpc;

grant create table to gzmpc;

grant unlimited tablespace to gzmpc;

-- 最后修改一下密码
ALTER USER gzmpc IDENTIFIED BY gzmpc;
```

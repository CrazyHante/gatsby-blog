---
title: ORACLE 如何查询被锁定表及如何解锁释放session
subTitle: oracle
category: sql
cover: sql.png
---

```sql
--锁表查询SQL
SELECT object_name, machine, s.sid, s.serial# 
FROM gv$locked_object l, dba_objects o, gv$session s 
WHERE l.object_id　= o.object_id 
AND l.session_id = s.sid; 

--释放SESSION SQL: 
alter system kill session 'sid, serial#'; 

```

ora-01031:insufficint privileges错误
oracle 有两种权限:
一种是系统权限,一种是oracle自己的权限.
简单说就是:没权限!
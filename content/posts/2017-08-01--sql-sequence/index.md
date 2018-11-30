---
title: 查询sql自增序列
subTitle: oracle
category: sql
cover: sql.png
---

查询自增序列:

```sql
select sequence_name from ALL_SEQUENCES where sequence_name like '%xxxx%'
```




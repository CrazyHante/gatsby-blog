---
titile: 查询sql自增序列
subTitle: oracle
category: Diary
cover: 5.png
---

查询自增序列:

```sql
select sequence_name from ALL_SEQUENCES where sequence_name like '%xxxx%'
```




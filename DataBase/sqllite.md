[TOC]


最大的特色就是指令前加.；而pg是加\
```sql
#sqlite3 demo.db   
sqlite> .exit

#查看表结构
sqlite> .schema tableName

#查看有哪些表
sqlite> .tables
table_demo  
sqlite> select * from sqlite_master where type="table";
sqlite> select * from sqlite_master where type="table" and name="tableName";

```


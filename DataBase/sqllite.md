[TOC]

## ReadMe

sqlite各种版本的应用。

## 安装、配置

安装sqlite3

```bash
# apt-get install sqlite3
$ depend libsqlite3-0=3.7.13-1+deb7u2 but libsqlite3-0=3.7.13-1+deb7u3 is installed.
# apt-get install libsqlite3-0=3.7.13-1+deb7u2
```



## 指令
最大的特色就是指令前加.；而pg是加\
```sql
#sqlite3 demo.db   
sqlite> .h

#查看有哪些表
sqlite> .tables
table_demo  

#查看表结构
sqlite> .schema tableName

sqlite> select * from sqlite_master where type="table";
sqlite> select * from sqlite_master where type="table" and name="tableName";

sqlite> .exit
sqlite> .q
```


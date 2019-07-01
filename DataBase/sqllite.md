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



## 其它

```cpp
NULL;
INTEGER;  //带符号的整数。
REAL;     //是一个浮点值。
TEXT;     //文本字符串。
BLOB;     //blob数据。BLOB (binary large object)，二进制大对象。
```



### 约束条件

```bash
not null;
unique;
primary key;
	#主键；
check;
	#check(keyname < 100)
	#如果条件值为 false，则记录违反了约束，且不能输入到表；
autoincrement;
	#自增长；
default; 
	#默认值；

KeyName integer PRIMARY KEY autoincrement;
	#primary key要放在autoincrement前。
```



## SQLite C API



### memleak

#### sqlite3_open系列

```cpp
int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
int sqlite3_open_v2(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb,         /* OUT: SQLite db handle */
  int flags,              /* Flags */
  const char *zVfs        /* Name of VFS module to use */
);
//不管成功与否都应该调用以下sqlite3_close()释放资源；
	int sqlite3_close(sqlite3*);
	int sqlite3_close_v2(sqlite3*);
```



#### sqlite3_prepare系列

```cpp
//sqlite3_prepare*系列函数
int sqlite3_prepare_v2(
  sqlite3 *db,            /* Database handle */
  const char *zSql,       /* SQL statement, UTF-8 encoded */
  int nByte,              /* Maximum length of zSql in bytes. */
  sqlite3_stmt **ppStmt,  /* OUT: Statement handle */
  const char **pzTail     /* OUT: Pointer to unused portion of zSql */
);
//当成功时*ppStmt被设置成!nullptr应该由以下函数释放，释放后不一定被保证=nullptr;
	int sqlite3_finalize(sqlite3_stmt *pStmt);
```



#### sqlite3_malloc, sqlite3_free

```cpp
void *sqlite3_malloc(int);
void *sqlite3_realloc(void*, int);
//返回的内存需要程序员调用如下api进行释放；
	void sqlite3_free(void*);
```



#### sqlite3_exec

```cpp
//封装了sqlite3_prepare_v2(), sqlite3_step(), and sqlite3_finalize()系列函数的操作；
int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);
//如果发生了错误,errmsg将被sqlite3_malloc()设置空间，需要sqlite3_free()进行释放；
	void sqlite3_free(void*);
```



#### sqlite3_errmsg系列

```cpp
int sqlite3_errcode(sqlite3 *db);
int sqlite3_extended_errcode(sqlite3 *db);
	//返回码易被后续的sqlite3 api调用扰乱；

const char *sqlite3_errmsg(sqlite3*);
const void *sqlite3_errmsg16(sqlite3*);
	//不需要程序员管理返回的内存，但返回的内存可能被后续sqlite3 api调用扰乱；

const char *sqlite3_errstr(int);
	//不需要程序员管理返回的内存；
```



#### sqlite3_get_table

```cpp
//向后兼容而存在的一个api，不推荐使用；（应该用sqlite3_exec()来查询）
int sqlite3_get_table(
  sqlite3 *db,          /* An open database */
  const char *zSql,     /* SQL to be evaluated */
  char ***pazResult,    /* Results of the query */
  int *pnRow,           /* Number of result rows written here */
  int *pnColumn,        /* Number of result columns written here */
  char **pzErrmsg       /* Error msg written here */
);
//在成功的case下，**pazResult需要调用如下接口释放；
	void sqlite3_free_table(char **result);
//在失败的case下，*pzErrmsg需要调用如下接口释放；
	void sqlite3_free(void*);
```




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
sqlite> .mode line

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

### exec & callback

sqlite3_exe在执行select时要求传入一个callback，

- callback的意思是，会先执行sql语句，然后将结果传递给callback，callback根据结果再进一步执行。
- callback<font color=red>查询集是n条记录时，调用n次</font>。
- callback返回0，则sqlite3_exec() 将继续执行查询；返回非0，则sqlite3_exec()将立即中断查询, 且返回 SQLITE_ABORT。
- 查询有个老版本的api，`sqlite3_get_table`;

原型如下：

```cpp
int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);

typedef int(*sqlite_callback)(void* para, int columenCount, char** columnValue, char** columnName);
	//para，由sqlite3_exec传入的参数指针，或者说是指针参数
	//columnCount，查询到的这一条记录由多少个字段（多少列）
	//columnValue，该参数是双指针，查询出来的数据都保存在这里，它是一个1维数组，每一个元素都是一个char*,是一个字段内容，所以这个参数就可以不是单字节，而是可以为字符串等不定长度的数值，用字符串表示，以'\0'结尾。
	//columnName，该参数是双指针，语columnValue是对应的，表示这个字段的字段名称，
```



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

int sqlite3_reset(sqlite3_stmt *pStmt); //重置一个准备语句对象到它的初始状态，然后准备被重新执行。
int sqlite3_clear_bindings(sqlite3_stmt*); //重置绑定的参数。
int sqlite3_step(sqlite3_stmt*); //执行有前面sqlite3_prepare创建的准备语句，返回如下：
	//SQLITE_BUSY 获取不到锁来执行准备语句。
	//SQLITE_DONE 执行成功。
	//SQLITE_ROW 准备语句执行结果有数据返回（如select），每当准备好一条记录则返回此。
	//SQLITE_ERROR 执行准备语句出错。
	//SQLITE_MISUSE 函数被不恰当的使用。
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



## 编码相关

建库、建表指定不了编码（一般编译为默认使用utf8，除非启用`SQLITE_ENABLE_ICU`）；
编码使用对应的api版本（如sqlite3_open16）；

sqlite db的字符串存储类型TEXT.

> TEXT. The value is a text string, stored using the database encoding (UTF-8, UTF-16BE or UTF-16LE).
> sqlite3_open() 默认是utf8
> sqlite3_open16() 默认是UTF-16 in the native byte order. 


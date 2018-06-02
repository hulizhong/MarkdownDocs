[TOC]

## ReadMe
```bash
# apt-cache search libmemcache
libmemcached-dev - C and C++ client library to the memcached server (development files)
# apt-get install libmemcached-dev
```


## 命令
指令说明，如下表

|指令|说明|
|:--|:-----|
|set |命令用于一个新的值,为一个新的或现有的键(key)设置一个值。
|add |命令用于为一个值(value)设置为一个新的键(key)。如果键(key)已经存在，那么它输出NOT_STORED。
|replace |命令用来替换现有键的值。如果该键不存在，那么它输出NOT_STORED
|append |命令是用来添加一些数据到现有键(key)。数据是存储在键的现有数据之后。
|prepend |命令用于添加一些数据到现有的键(key)。数据将存储在键的现有的数据之前。
|cas |命令用于设置数据，如果自上一次获取没有人更新。如果该键不在memcached中，那么它返回NOT_FOUND。
|get |命令用于获取存储在键的值。如果该键在memcached 中不存在，那么它没有返回值。
|gets |命令用于获取cas令牌值。如果该键在 memcached 中不存在，那么它没有返回值。
|delete |命令用于删除memcached服务器现有的键。
|flush_all |命令用于删除memcached服务器中的所有数据(键值对)。它接受一个叫做time可选参数，表示这个时间后的所有memcached数据会被清除。    


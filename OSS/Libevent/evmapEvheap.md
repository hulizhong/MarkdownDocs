[toc]

## ReadMe
io\_map, signal\_map这些结构体的定义
如何向这些结构增、删event

## io\_map的定义
```cpp
#ifdef EVMAP_USE_HT
	#define HT_NO_CACHE_HASH_VALUES
	#include "ht-internal.h"
	struct event_map_entry;
	HT_HEAD(event_io_map, event_map_entry);
#else
	//如果EVMAP_USE_HT没有被定义那么，io_map同于signal_map，只是key为fd
	#define event_io_map event_signal_map
#endif
```

用HashTable的场景下
```cpp
HT_HEAD(event_io_map, event_map_entry);

#define HT_HEAD(name, type)                                             \
  struct name {                                                         \
    /* The hash table itself. */                                        \
    struct type **hth_table;                                            \
    /* How long is the hash table? */                                   \
    unsigned hth_table_length;                                          \
    /* How many elements does the table contain? */                     \
    unsigned hth_n_entries;                                             \
    /* How many elements will we allow in the table before resizing it? */ \
    unsigned hth_load_limit;                                            \
    /* Position of hth_table_length in the primes table. */             \
    int hth_prime_idx;                                                  \
  }

//那么经过宏替换之后应该是如下的：
struct event_io_map {
	struct event_map_entry **hth_table; //hash table
	unsigned hth_table_length; //hash table的长度
	unsigned hth_n_entries; //hash table的元素个数
	unsigned hth_load_limit; //table空间重分配之前，能够装的元素个数
	int hth_prime_idx; //用到了哪个素数
}

//其中的event_map_entry
struct event_map_entry {
	HT_ENTRY(event_map_entry) map_node;
	evutil_socket_t fd;
	union { /* This is a union in case we need to make more things that can
			   be in the hashtable. */
		struct evmap_io evmap_io;
	} ent;
};

#define HT_ENTRY(type)                          \
  struct {                                      \
    struct type *hte_next;                      \
    unsigned hte_hash;                          \
  }

struct event_io_map {
	struct event_map_entry {
		struct {
			struct event_map_entry *hte_next;
			unsigned hte_hash;  //对存储元素进行hash算法时，模值；（是从fd计算过来的）
		}map_node;
		evutil_socket_t fd;
		union {
			struct evmap_io evmap_io;
		} ent;
	}**hth_table; //hash table
	unsigned hth_table_length; //hash table的长度
	unsigned hth_n_entries; //hash table的元素个数
	unsigned hth_load_limit; //table空间重分配之前，能够装的元素个数
	int hth_prime_idx; //用到了哪个素数
}

```

### hash table的额外知识
很多查找方法都是基于待查关键字与表中元素进行比较而实现的；
> 如此查找的效率依赖于查找过程中所比较的次数；
> 理想的情况是不通过任何比较，而一次便能得到所查的记录；
>> 既是一种查找方法，又是一种存贮方法；


HashTable散列存储结构
> 基本思想
>> 建立关键码字与其存储位置的对应关系；
>
> 优点
>> 查找速度快O(1)，查找效率与元素个数n无关；


哈希方法需要解决的问题
> 构造好的哈希函数
>> 函数要尽可能简单，以便提高转换速度；  
>> 函数对关键码计算出的地址，应在哈希地址集中大致均匀分布，以减少空间浪费；
>
> 制定一个好的解决冲突的方案
>> hash函数计算出的地址之前已经存储过数据，那么如何存储这个新元素呢？  
>> 对应查找也是一样的；  
>


- 解决冲突的方法
	- 链地址法，将具有相同hash地址的记录链成一个单链表；
		- hash地址就设m个单链表，然后用数组将m个链表表头指针存储起来，形成一个动态的结构；
	- 开放地址法，冲突时寻找下一个空的哈希地址（只要hash table够大，空的位置总能找到）
		- H=(Hash(key)+d) mod m
			- d为增量序列>=1,<m；
		- 优点
			- 只要hash table未填满，就能找到存储的地；
		- 缺点
			- 第i个位置被冲突元素占了，本来hash到这个位置的元素就只能靠后，降低查找效率；

- hash函数的构造方法
	- 取模% Hask(key) = key mod p
		- p是一个整数，一般选p<=哈希空间m，且为质数；
	- 等等


Hash表的查找效率探索
> 因为冲突的存在，使得hash table的查找过程仍然要进行比较，需要用到ASL来衡量；
>> 平均查找长度ASL依赖哈希表的装填因子（装满程度） a=表中的记录数/哈希表的空间
>>> 链地址法 ASL=1+a/2
>>> 开地址法 ASL=(1/2)(1+(1/(1-a)))


## signal\_map的定义
```cpp
//用于信号到一系列event的映射；
struct event_signal_map {
	// 一个evmap_io*或者evmap_signal*的数组；没有元素则为NULL
	void **entries;
	int nentries; //entries数组的大小；
};

struct evmap_signal {
	struct event_dlist {
		struct event *lh_first; //first element
	}events;
};
```

event\_dlist的扩展过程
```cpp
LIST_HEAD (event_dlist, event); 

struct event_dlist {
	struct event *lh_first; //first element
}

#define LIST_HEAD(name, type)						\
struct name {								\
	struct type *lh_first;  /* first element */			\
	}
```

### 被始化、清空
evmap\_signal\_initmap\_
evmap\_signal\_clear\_
```cpp
void evmap_signal_initmap_(struct event_signal_map *ctx)
{
	ctx->nentries = 0;
	ctx->entries = NULL;
}

void evmap_signal_clear_(struct event_signal_map *ctx)
{
	if (ctx->entries != NULL) {
		int i;
		for (i = 0; i < ctx->nentries; ++i) {
			if (ctx->entries[i] != NULL)
				mm_free(ctx->entries[i]);
		}
		mm_free(ctx->entries);
		ctx->entries = NULL;
	}
	ctx->nentries = 0;
}
```

### 增加、删除
evmap\_signal\_add\_
evmap\_signal\_del\_
```cpp
int evmap_signal_add_(struct event_base *base, int sig, struct event *ev)
{
	const struct eventop *evsel = base->evsigsel;
	struct event_signal_map *map = &base->sigmap;
	struct evmap_signal *ctx = NULL;

	if (sig >= map->nentries) {
		if (evmap_make_space(
			map, sig, sizeof(struct evmap_signal *)) == -1)
			return (-1);
	}
	GET_SIGNAL_SLOT_AND_CTOR(ctx, map, sig, evmap_signal, evmap_signal_init,
	    base->evsigsel->fdinfo_len);

	if (LIST_EMPTY(&ctx->events)) {
		if (evsel->add(base, ev->ev_fd, 0, EV_SIGNAL, NULL)
		    == -1)
			return (-1);
	}

	LIST_INSERT_HEAD(&ctx->events, ev, ev_signal_next);

	return (1);
}

int evmap_signal_del_(struct event_base *base, int sig, struct event *ev)
{
	const struct eventop *evsel = base->evsigsel;
	struct event_signal_map *map = &base->sigmap;
	struct evmap_signal *ctx;

	if (sig >= map->nentries)
		return (-1);

	GET_SIGNAL_SLOT(ctx, map, sig, evmap_signal);

	LIST_REMOVE(ev, ev_signal_next);

	if (LIST_FIRST(&ctx->events) == NULL) {
		if (evsel->del(base, ev->ev_fd, 0, EV_SIGNAL, NULL) == -1)
			return (-1);
	}

	return (1);
}

```

### 激活
evmap\_signal\_active\_
```cpp
void evmap_signal_active_(struct event_base *base, evutil_socket_t sig, int ncalls)
{
	struct event_signal_map *map = &base->sigmap;
	struct evmap_signal *ctx;
	struct event *ev;

	if (sig < 0 || sig >= map->nentries)
		return;
	GET_SIGNAL_SLOT(ctx, map, sig, evmap_signal);

	if (!ctx)
		return;
	LIST_FOREACH(ev, &ctx->events, ev_signal_next)
		event_active_nolock_(ev, EV_SIGNAL, ncalls);
}
```

## LIST\_EMPTY


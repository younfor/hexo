---
title: Redis实战
date: 2018-06-11 12:55:17
categories: Redis
tags: [Redis, 技术栈]
---
# 应用场景
* 当做缓存，缓存未命中再访问MySQL，这样能降低读的延迟
* 原子计数
* 最新TopN数据
使用list中的LTRIM latest.comments 0 5000 这样永远只保存最近的5000个ID
* 带权值的排序
使用sorted set， 然后ZADD命令
* 过期时间
* 优先级的队列系统

# 数据类型
* List
* Set
* Sorted set
* Hash
* 自增原子
# 特点
* 主从同步
* 持久化
# strings
```
struct sdshdr {
    long len;
    long free;
    char buf[];
}
```
* set
```
set name "hello"
```
* setnx
其中nx是不存在的意思, 如果存在则设置失败
```
setnx name "hello"
```
* setex
如果超过10秒，get就会为空
```
setex name 10 "red"
```
* setrange
替换字符串
* mset 
设置多组值
* msetnx
会回滚
* getset
返回旧值
* mget
获取多组数据，如果不存在返回nil
* incr/decr
自增，如果不存在默认为1,而且用get的时候得到的会是字符串
# hashes
* hset
```
hset myhash filed1 hello
```
* hsetnx
如果key不存在则创建，如果key存在则返回0（不会覆盖）
* hmset
设置多个filed
* hget
```
hget myhash field1
```
* hmget
* hincrby
```
// 给指定的filed加定值
hincrby myhash field3 -8
```
* hexists
```
//判断filed是否存在
hexists myhash filed1
``` 
* hlen 
返回filed的数量
* hdel
删除指定filed，并返回1 成功
* hkeys hvals hgetall
# lists
链表结构，主要只有push和pop, 是一个双向链表，长度为2的32次方，既可以当做栈也可以当做队列
也有阻塞版本的pop
* lpush rpush
```
// l和r是表示左右的意思
lpush mylist "world"
lrange mylist 0 -1
```
* linsert
```
// 在world前面加上there
linsert mylist3 before "world" "there"
```
* lset
```
设置指定下标的值
lset mylist 0 "four"
```
* lrem
```
// 从 key 对应 list 中删除 count 个和 value 相同的元素
// count > 0 删除count， count < 0 从尾部开始删除, count == 0删除所有
lrem mylist 2 "hello"
```
* ltrim
```
// 只保留from到to的数据
ltrim mylist8 1 -1 // 相当于删除了第一个元素
```
* lpop
删除头部元素并返回头部元素
* rpoplpush
原子操作，从第一个链表取出来放到第二个链表头，并返回这个值
* lindex 
```
lindex mylist index  // 返回第index的位置
```
# sets
set 是集合， hashtable实现， sorted_set是跳表
* sadd
* smemebers
获取所有元素
* srem
删除
* spop
随机返回
* sdiff
去除 a里面和b相关，把剩下的返回
比如  1 2 3 diff 3 4 5 =  1 2
* sdiffstore
```
// 与sdiff的区别就是会存起来
sdiffstore myset1 myset2 tomyset3
``` 
* sinter sinterstore
交集
* sunion sunionstore
并集
* smove
```
smove mysetfrom mysetto three
// 把three从from移动到to
```
* scard
返回元素个数
* sismember
返回是否是元素
* srandmember 
随机返回，不删除
# sorted sets
排序的， 跳表+hashtable
```
zadd myzset 1 "one"
zadd myzset 2 "two"
zadd myzset 3 "three"
zrange myzset 0 -1 withscores // zrevrange是从大到小的顺序
// "one" "1" "two" "2" "three" "3"
```
* zincrby
```
zincrby myset 2 "one" // 增加分数
```
* zrank zrevrank
从小到大返回 下标, 从大到小返回 下标
* zrangebyscore
* zcount 
返回分数score区间的个数
* zcard
返回集合个数
* zscore 
返回元素的score
* zremrangebyrank
删除排名区间的， 下标顺序
* zremrangebyscore 
删除score区间的元素

* keys *
列出所有key, 可以用字符串匹配 keys abc* 之类
* exists
判断key是否存在
* del 
删除key
* expire 
设置有效时间, 通过ttl获取有效时长
* move 
转移数据库
```
select 0
set age 30
move age 1
select 1
get age
```
* persist
移除过期时间
* randomkey
* rename
* type
获取key对应的value类型
* ping  info  quit ..

#应用场景
* 当做缓存，缓存未命中再访问MySQL，这样能降低读的延迟
* 原子计数
* 最新TopN数据
使用list中的LTRIM latest.comments 0 5000 这样永远只保存最近的5000个ID
* 带权值的排序
使用sorted set， 然后ZADD命令
* 过期时间
* 优先级的队列系统

#数据类型
* List
* Set
* Sorted set
* Hash
* 自增原子
#特点
* 主从同步
* 持久化
#strings
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
#hashes

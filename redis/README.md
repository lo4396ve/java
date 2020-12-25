---
title: 2020-12-21
date: 2020-12-21 14:24
---
# Redis
## 介绍
redis[官方介绍](https://redis.io/topics/introduction)。
## 学习 redis 的理由
* redis 很牛，即使用不到，也应该学习了解
* 性能好，适合一些场景做缓存，高并发
* 面试常问（大多问原理）
## 常用数据类型
常用的数据类型有 String，Hash，List，Set，SortedSet(ZSet)
### 1.数据类型-String
    String 可以存字符串也可以存数字
#### SET key value [EX seconds|PX milliseconds|KEEPTTL] [NX|XX] [GET] 
    将键key设定为指定的“字符串”值，如果 key 已经保存了一个值，那么这个操作会直接覆盖原来的值，并且忽略原始类型，当set命令执行成功之后，之前设置的过期时间都将失效
**Options**
* EX seconds – 设置键key的过期时间，单位时秒
* PX milliseconds – 设置键key的过期时间，单位时毫秒
* NX – 只有键key不存在的时候才会设置key的值
* XX – 只有键key存在的时候才会设置key的值
**注意：**
    由于SET命令加上选项已经可以完全取代SETNX, SETEX, PSETEX的功能，所以在将来的版本中，redis可能会不推荐使用并且最终抛弃这几个命令。

```
redis> SET mykey "Hello"
OK
redis> GET mykey
"Hello"
```
#### SETEX key seconds value
    设置key对应字符串value，并且设置key在给定的seconds时间之后超时过期。
    等效于以下命令组合：
        SET mykey value
        EXPIRE mykey seconds    // 对 key 设置过期时间
```
redis> SETEX mykey 10 "Hello"
OK
redis> TTL mykey    // 获取 key 的时效时长
(integer) 10
redis> GET mykey
"Hello"
```
#### SETNX key value
    如果key不存在，将key设置值为value，当key存在时，什么也不做。SETNX是”SET if Not eXists”的简写。
    如果 key 被设置了返回 1，如果没有被设置返回 0
```
redis> SETNX mykey "Hello"
(integer) 1
redis> SETNX mykey "World"
(integer) 0
redis> GET mykey
"Hello"
```

#### GET key
    返回key的value。如果key不存在，返回特殊值nil。如果key的value不是string，就返回错误，因为GET只处理string类型的values。
```
redis> SET mykey "Hello"
OK
redis> GET mykey
"Hello"
redis> GET mykey1
(nil)
```
#### GETSET key value
    自动将key对应到value并且返回原来key对应的value，如果之前Key不存在将返回nil，如果key存在但是对应的value不是字符串，就返回错误。
```
redis> SET mycounter “0”
(integer) 1
redis> GETSET mycounter “1”
“0”
redis> GET mycounter
“1”
```
#### APPEND key value
    如果 key 已经存在，并且值为字符串，那么这个命令会把 value 追加到原来值（value）的结尾。 如果 key 不存在，那么它将首先创建一个空字符串的key，再执行追加操作，这种情况 APPEND 将类似于 SET 操作。返回操作后字符串值（value）的长度。
```
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey " World"
(integer) 11
redis> GET mykey
"Hello World"
```
#### STRLEN key
    返回key的string类型value的长度。如果 key 不存在，返回 0，如果key对应的非string类型，就返回错误。
```
redis> SET mykey "Hello world"
OK
redis> STRLEN mykey
(integer) 11
redis> STRLEN nonexisting
(integer) 0
```
#### MSET key value [key value ...] 
    对应给定的keys到他们相应的values上。MSET会用新的value替换已经存在的value，就像普通的SET命令一样。总是 返回OK，因为MSET不会失败。
```
redis> MSET key1 "Hello" key2 "World"
OK
redis> GET key1
"Hello"
redis> GET key2
"World"
```
#### MGET key [key ...] 
    返回所有指定的key的value。对于每个不对应string或者不存在的key，都返回特殊值nil。正因为此，这个操作从来不会失败。
```
redis> SET key1 "Hello"
OK
redis> SET key2 "World"
OK
redis> MGET key1 key2 nonexisting
1) "Hello"
2) "World"
3) (nil)
```
#### INCR key
    对存储在指定key的数值执行原子的加1操作。返回执行递增操作后key对应的值。
**注意：**
1. 如果指定的key不存在，那么在执行incr操作之前，会先将它的值设定为0。
2. 如果指定的key中存储的值不是字符串类型（fix：）或者存储的字符串类型不能表示为一个整数，返回一个错误(eq:(error) ERR value is not an integer or out of range)。
3. 这个操作仅限于64位的有符号整型数据。
```
redis> SET mykey "10"
OK
redis> INCR mykey
(integer) 11
redis> GET mykey
"11"
```
#### INCRBY key increment
    将key对应的数字加decremen，返回增加之后的value值。
**注意点同上**
```
redis> SET mykey "10"
OK
redis> INCRBY mykey 5
(integer) 15
```
#### DECR key
    对key对应的数字做减1操作，返回减小之后的value
**注意点同上**
```
redis> SET mykey "10"
OK
redis> DECR mykey
(integer) 9
```
#### DECRBY key decrement
    将key对应的数字减decrement，返回减小之后的value
**注意点同上**
```
redis> SET mykey "10"
OK
redis> DECRBY mykey 5
(integer) 5
```
#### INCRBYFLOAT key increment
    通过指定浮点数key来增长浮点数(存放于string中)的值. 当键不存在时,先将其值设为0再操作。如果操作命令成功, 相加后的值将替换原值存储在对应的键值上, 并以string的类型返回. string中已存的值或者相加参数可以任意选用指数符号,但相加计算的结果会以科学计数法的格式存储. 无论各计算的内部精度如何, 输出精度都固定为小数点后17位。
**注意以下情况导致返回错误：**
1. key 包含非法值(不是一个string).
2. 当前的key或者相加后的值不能解析为一个双精度的浮点值.(超出精度范围了)
```
redis> SET mykey 10.50
OK
redis> INCRBYFLOAT mykey 0.1
"10.6"
redis> SET mykey 5.0e3
OK
redis> INCRBYFLOAT mykey 2.0e2
"5200"
```

### 2.数据类型 - Hash散列类型
可以存序列化 JSON 对象
存储方式
键(key)-字段(field)-字段值(value)
User-name-张三
User-age-18
User-sex-woman

#### HSET key field value 
设置 key 指定的哈希集中指定字段的值，如果 key 指定的哈希集不存在，会创建一个新的哈希集并与 key 关联，如果字段在哈希集中存在，它将被重写。
如果field是一个新的字段返回 1，如果field原来在map里面已经存在返回 0
```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HGET myhash field1
"Hello"
```
#### HMSET key field value [field value ...] 
    设置多个字段值
```
redis> HMSET myhash field1 "Hello" field2 "World"
OK
redis> HGET myhash field1
"Hello"
redis> HGET myhash field2
"World"
```
#### HSETNX key field value
    只在 key 指定的哈希集中不存在指定的字段时，设置字段的值，否则不执行任何操作。如果字段是个新的字段并成功赋值返回 1，如果哈希集中已存在该字段没有操作执行返回 0
```
redis> HSETNX myhash field "Hello"
(integer) 1
redis> HSETNX myhash field "World"
(integer) 0
redis> HGET myhash field
"Hello"
```
#### HGET key field
    返回 key 指定的哈希集中该字段所关联的值，当字段不存在或者 key 不存在时返回nil。
```
redis> HSET myhash field1 "foo"
(integer) 1
redis> HGET myhash field1
"foo"
redis> HGET myhash field2
(nil)
```
#### HMGET key field [field ...] 
    返回 key 指定的哈希集中指定字段的值，对于哈希集中不存在的每个字段，返回 nil 值。
```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HMGET myhash field1 field2 nofield
1) "Hello"
2) "World"
3) (nil)
```
#### HGETALL key
    返回 key 指定的哈希集中所有的字段和值。当 key 指定的哈希集不存在时返回空列。
```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HGETALL myhash
1) "field1"
2) "Hello"
3) "field2"
4) "World"
```
#### HEXISTS key field
    返回hash里面field是否存在。存在返回 1，不存在返回 0
```
redis> HSET myhash field1 "foo"
(integer) 1
redis> HEXISTS myhash field1
(integer) 1
redis> HEXISTS myhash field2
(integer) 0
```

#### HKEYS key
    返回 key 指定的哈希集中所有字段的名字。
```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HKEYS myhash
1) "field1"
2) "field2"
```
#### HVALS key
    返回 key 指定的哈希集中所有字段的值。
```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HVALS myhash
1) "Hello"
2) "World"
```
#### HLEN key
    返回 key 指定的哈希集包含的字段的数量。当 key 指定的哈希集不存在时返回 0。
```
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HLEN myhash
(integer) 2
```
#### HDEL key field [field ...] 
    从 key 指定的哈希集中移除指定的域。在哈希集中不存在的域将被忽略。返回从哈希集中成功移除的 field的数量
```
redis> HSET myhash field1 "foo"
(integer) 1
redis> HDEL myhash field1
(integer) 1
redis> HDEL myhash field2
(integer) 0
```

### 3.数据类型 - List
Redis的list是采用来链表来存储的，特点：增删快、查询慢，但是查询链表两端的数据也很快。
对于redis的list数据类型的操作，是操作list的两端数据来操作的。
#### LPUSH key value [value ...] 
   将所有指定的值插入到存于 key 的列表的头部。如果 key 不存在，那么在进行 push 操作前会创建一个空列表。 在 push 操作后的 list 长度，如果 key 对应的值不是一个 list 的话，那么会返回一个错误。
**注意：**
    LPUSH mylist a b c，返回的列表是 c 为第一个元素， b 为第二个元素， a 为第三个元素。
```
redis> LPUSH mylist "world"
(integer) 1
redis> LPUSH mylist "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```
#### LPUSHX key value
    只有当 key 已经存在并且存着一个 list 的时候，在这个 key 下面的 list 的头部插入 value。当 key 不存在的时候不会进行任何操作。返回在 push 操作后的 list 长度。
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSHX mylist "Hello"
(integer) 2
redis> LPUSHX myotherlist "Hello"
(integer) 0
redis> LRANGE mylist 0 -1
1) "Hello"
2) "World"
redis> LRANGE myotherlist 0 -1
(empty list or set)
```

#### RPUSH key value [value ...] 
    向存于 key 的列表的尾部插入所有指定的值。如果 key 不存在，那么会创建一个空的列表然后再进行 push 操作。 当 key 保存的不是一个列表，那么会返回一个错误。
**注意**
    RPUSH mylist a b c 会返回一个列表，其第一个元素是 a ，第二个元素是 b ，第三个元素是 c。
```
redis> RPUSH mylist "hello"
(integer) 1
redis> RPUSH mylist "world"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```
#### RPUSHX key value
    将值 value 插入到列表 key 的表尾, 当且仅当 key 存在并且是一个列表。 和 RPUSH 命令相反, 当 key 不存在时不会进行任何操作。返回在 push 操作后的 list 长度。
```
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSHX mylist "World"
(integer) 2
redis> RPUSHX myotherlist "World"
(integer) 0
redis> LRANGE mylist 0 -1
1) "Hello"
2) "World"
redis> LRANGE myotherlist 0 -1
(empty list or set)
```
#### LRANGE key start stop
    返回存储在 key 的列表里指定范围内的元素，start 和 end 偏移量都是基于0的下标，-1 表示列表的最后一个元素，-2 是倒数第二个。
**注意：**
    当下标超过list范围的时候不会产生error。 如果start比list的尾部下标大的时候，会返回一个空列表。 如果stop比list的实际尾部大的时候，Redis会当它是最后一个元素的下标。
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LRANGE mylist 0 0
1) "one"
redis> LRANGE mylist -3 2
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist -100 100
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist 5 10
(empty list or set)
```
#### LPOP key
    移除并且返回 key 对应的 list 的第一个元素。或者当 key 不存在时返回 nil。
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LPOP mylist
"one"
redis> LRANGE mylist 0 -1
1) "two"
2) "three"
```
#### RPOP key
    移除并返回存于 key 的 list 的最后一个元素。或者当 key 不存在的时候返回 nil。
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> RPOP mylist
"three"
redis> LRANGE mylist 0 -1
1) "one"
2) "two"
```
#### LLEN key
    返回存储在 key 里的list的长度，如果 key 不存在，那么就被看作是空list，并且返回长度为 0。 当存储在 key 里的值不是一个list的话，会返回error。
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LLEN mylist
(integer) 2
```
#### LREM key count value
    从存于 key 的列表里移除前 count 次出现的值为 value 的元素。返回被移除的元素个数。
**Options**
* count
    * count > 0: 从头往尾遍历移除值为 value 的元素。
    * count < 0: 从尾往头移除值为 value 的元素。
    * count = 0: 移除所有值为 value 的元素
**注意：**
    如果list里没有存在key就会被当作空list处理，所以当 key 不存在的时候，这个命令会返回 0。
```
redis> RPUSH mylist "hello"
(integer) 1
redis> RPUSH mylist "hello"
(integer) 2
redis> RPUSH mylist "foo"
(integer) 3
redis> RPUSH mylist "hello"
(integer) 4
redis> LREM mylist -2 "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "foo"
```
LINDEX key index
    获得指定索引的元素值，index是从0开始索引的，-1 表示最后一个元素，-2 表示倒数第二个元素。index 超出范围返回 nil
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LINDEX mylist 0
"Hello"
redis> LINDEX mylist -1
"World"
redis> LINDEX mylist 3
(nil)
```
#### LINSERT key BEFORE|AFTER pivot value
    把 value 插入存于 key 的列表中在基准值 pivot 的前面或后面，当 key 不存在时，这个list会被看作是空list，任何操作都不会发生，当 key 存在，但保存的不是一个list的时候，会返回error。
    返回经过插入操作后的list长度，当 pivot 值找不到的时候返回 -1。
```
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
```
#### LTRIM key start stop
    修剪(trim)一个已存在的 list，这样 list 就会只包含指定范围的指定元素。start 和 stop 都是由0开始计数的， 这里的 0 是列表里的第一个元素（表头），1 是第二个元素，-1 是最后一个元素，以此类推。
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LTRIM mylist 1 -1
OK
redis> LRANGE mylist 0 -1
1) "two"
2) "three"
```
#### BLPOP key [key ...] timeout
    BLPOP 是阻塞式列表的弹出原语。 它是命令 LPOP 的阻塞版本，[参考官方 API](http://www.redis.cn/commands/blpop.html)
#### BRPOP key [key ...] timeout
    BRPOP 是一个阻塞的列表弹出原语。 它是 RPOP 的阻塞版本，[参考官方 API](http://www.redis.cn/commands/brpop.html)

### 4. 数据类型 - Set
>  特点：Set类型：无序、不可重复 List类型：有序、可重复
#### SADD key member [member ...] 
    添加一个或多个指定的member元素到集合的 key中.指定的一个或者多个元素member 如果已经在集合key中存在则忽略.如果集合key 不存在，则新建集合key,并添加member元素到集合key中。
    返回新成功添加到集合里元素的数量，不包括已经存在于集合中的元素，如果key 的类型不是集合则返回错误。
```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SADD myset "World"
(integer) 0
redis> SMEMBERS myset    // 返回key集合所有的元素
1) "World"
2) "Hello"
```
#### SCARD key
    返回集合存储的key的基数 (集合元素的数量)
```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SCARD myset
(integer) 2
```
#### SDIFF key [key ...] 
    返回一个集合与给定集合的差集的元素。
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFF key1 key2
1) "a"
2) "b"
```
#### SINTER key [key ...] 
    返回指定所有的集合的成员的交集
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTER key1 key2
1) "“c”
```
#### SUNION key [key ...] 
    返回给定的多个集合的并集中的所有成员。
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SUNION key1 key2
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
```
#### SDIFFSTORE destination key [key ...] 
    该命令类似于 SDIFF, 不同之处在于该命令不返回结果集，而是将结果存放在destination集合中，如果destination已经存在, 则将其覆盖重写。
    返回结果集元素的个数。
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFFSTORE key key1 key2
(integer) 2
redis> SMEMBERS key
1) "b"
2) "a"
```
#### SINTERSTORE destination key [key ...] 
    这个命令与SINTER命令类似, 但是它并不是直接返回结果集,而是将结果保存在 destination集合中。如果destination 集合存在, 则会被重写。
    返回结果集中成员的个数
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTERSTORE key key1 key2
(integer) 1
redis> SMEMBERS key
1) "c"
```
#### SUNIONSTORE destination key [key ...] 
    该命令作用类似于SUNION命令,不同的是它并不返回结果集,而是将结果存储在destination集合中。如果destination 已经存在,则将其覆盖。
    返回结果集中元素的个数。
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SUNIONSTORE key key1 key2
(integer) 5
redis> SMEMBERS key
1) "c"
2) "e"
3) "b"
4) "a"
5) "d"
```
#### SISMEMBER key member 
    返回成员 member 是否是存储的集合 key的成员
```
redis> SADD myset "one"
(integer) 1
redis> SISMEMBER myset "one"
(integer) 1
redis> SISMEMBER myset "two"
(integer) 0
```
#### SMEMBERS key
    返回key集合所有的元素。
```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SMEMBERS myset
1) "World"
2) "Hello"
```
#### SMOVE source destination member
    将member从source集合移动到destination集合中。如果该元素成功移除,返回1，如果该元素不是 source集合成员,无任何操作,则返回0。
**注意：**
    如果source 集合不存在或者不包含指定的元素,这smove命令不执行任何操作并且返回0.否则对象将会从source集合中移除，并添加到destination集合中去，如果destination集合已经存在该元素，则smove命令仅将该元素充source集合中移除. 如果source 和destination不是集合类型,则返回错误
```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myotherset "three"
(integer) 1
redis> SMOVE myset myotherset "two"
(integer) 1
redis> SMEMBERS myset
1) "one"
redis> SMEMBERS myotherset
1) "three"
2) "two"
```
#### SPOP key [count] 
    从存储在key的集合中移除并返回 count个随机元素。返回被删除的元素，当key不存在时返回nil。
**注意：**
1. count参数将在更高版本中提供，但是在2.6、2.8、3.0中不可用。Redis 3.2是第一个可以给SPOP传递可选参数count的版本
2. 如果count大于集合内部的元素数量，此命令将会返回整个集合，不会有额外的元素。
3. 如果不传 count，相当于 count 传 1
```
redid> SADD myset "one"
(integer) 1
127.0.0.1:6379> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SPOP myset
"three"
redis> SADD myset "four"
(integer) 1
redis> sadd myset "five"
(integer) 1
redis> SMEMBERS myset
1) "one"
2) "five"
3) "four"
4) "two"
```
#### SRANDMEMBER key [count] 
    随机返回key集合中的一个 或几个元素。
**注意：**
1. Redis 2.6开始，可以接受 count 参数
2. 如果count是整数且小于元素的个数，返回含有 count 个不同的元素的数组
3. 如果count是个整数且大于集合中元素的个数时，仅返回整个集合的所有元素
4. 当count是负数，则会返回一个包含count的绝对值的个数元素的数组，如果count的绝对值大于元素的个数，则返回的结果集里会出现一个元素出现多次的情况
5. 不提供 count 参数，相当于 count=1
**返回值：**
* 不使用count 参数的情况下该命令返回随机的元素，如果key不存在则返回nil
* 使用count参数,则返回一个随机的元素数组，如果key不存在则返回一个空的数组。
```
redis> SADD myset one two three
(integer) 3
redis> SRANDMEMBER myset
"one"
redis> SRANDMEMBER myset 2
1) "three"
2) "one"
redis> SRANDMEMBER myset -5
1) "one"
2) "one"
3) "one"
4) "one"
5) "one"
```
#### SREM key member [member ...] 
    在key集合中移除指定的元素. 如果指定的元素不是key集合中的元素则忽略，返回从集合中移除元素的个数，不包括不存在的成员， 如果key集合不存在则被视为一个空的集合，该命令返回0。
**注意：**
    Redis 2.4 之前的版本每次只能移除一个元素，2.4 以后接受多个 member 元素参数```

```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SREM myset "one"
(integer) 1
redis> SREM myset "four"
(integer) 0
redis> SMEMBERS myset
1) "three"
2) "two"
```

### 数据类型 - Sortedset (ZSet)
>  ZSet特点：有序，不可重复；Set特点：无序，不可重复
**说明（抄自菜鸟教程）：**
    Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。
    不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。
    有序集合的成员是唯一的,但分数(score)却可以重复。

#### ZADD key [NX|XX] [CH] [INCR] score member [score member ...] 
    将所有指定成员添加到键为key有序集合（如果key不存在，将会创建一个新的有序集合）里面。 添加时可以指定多个分数/成员（score/member）对。 如果指定添加的成员已经存在，则会更新改成员的分数（scrore）并更新到正确的排序位置。
**Options**
    ZADD 命令在key后面分数/成员（score/member）对前面支持一些参数（(>= Redis 3.0.2)）
        * XX: 仅仅更新存在的成员，不添加新成员
        * NX: 不更新存在的成员。只添加新成员 
        * CH: 修改返回值为发生变化的成员总数，原始是返回新添加成员的总数 (CH 是 changed 的意思)。更改的元素是新添加的成员，已经存在的成员更新分数。 所以在命令中指定的成员有相同的分数将不被计算在内。注：在通常情况下，ZADD返回值只计算新添加成员的数量。
        * INCR: 当ZADD指定这个选项时，成员的操作就等同ZINCRBY命令，对成员的分数进行递增操作
**相同分数的成员排序规则**
    * 当多个成员有相同的分数时，他们将是有序的字典（ordered lexicographically）（仍由分数作为第一排序条件，然后，相同分数的成员按照字典规则相对排序）
    * 字典顺序排序用的是二进制，它比较的是字符串的字节数组
    * 如果用户将所有元素设置相同分数（例如0），有序集合里面的所有元素将按照字典顺序进行排序
**返回值**
        * 如果不提供选填参数，返回添加到有序集合的成员数量，不包括已经存在更新分数的成员。
        * 如果指定 CH 参数，返回发生变化的成员总数。
        * 如果指定INCR参数，返回成员的新分数（双精度的浮点型数字）字符串。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 1 "uno"
(integer) 1
redis> ZADD myzset 2 "two" 3 "three"
(integer) 2
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "uno"
4) "1"
5) "two"
6) "2"
7) "three"
8) "3"
```
#### ZRANK key member 
    返回有序集key中成员member的排名。其中有序集成员按score值递增(从小到大)顺序排列。排名以0为底，也就是说，score值最小的成员排名为0。    如果member不是有序集key的成员，返回 nil。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZRANK myzset "three"
(integer) 2
redis> ZRANK myzset "four"
(nil)
```
#### ZREVRANK key member
    使用ZRANK命令可以获得成员按score值递增(从小到大)排列的排名。
    返回有序集key中成员member的排名，其中有序集成员按score值从大到小排列。排名以0为底，也就是说，score值最大的成员排名为0。
    如果member不是有序集key的成员，返回nil。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREVRANK myzset "one"
(integer) 2
redis> ZREVRANK myzset "four"
(nil)
redis>
```
#### ZCARD key
    返回key的有序集元素个数。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZCARD myzset
(integer) 2
redis>
```
#### ZINCRBY key increment member
    为有序集key的成员member的score值加上增量increment。如果key中不存在member，就在key中添加一个member，如果key不存在，就创建一个只含有指定member成员的有序集合。
    当key不是有序集类型时，返回一个错误。
    score值必须是字符串表示的整数值或双精度浮点数，并且能接受double精度的浮点数。也有可能给一个负数来减少score的值。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZINCRBY myzset 2 "one"
"3"
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "one"
4) "3"
redis>
```

#### ZRANGE key start stop [WITHSCORES] 
    返回存储在有序集合key中的指定范围的元素。 返回的元素可以认为是按得分从最低到最高排列。 如果得分相同，将按字典排序。如果指定了WITHSCORES选项，将同时返回它们的得分。
**注意：**
    * 参数start和stop都是基于零的索引，可以是负数，0是第一个元素，1是第二个元素，-1是最后一个元素
    * start和stop都是全包含的区间，例如ZRANGE myzset 0 1 会返回第一个和第二个元素
    * 超出范围的索引不会产生错误，如果 start 大于有序集合最大索引或者start > stop 返回空列表，如果stop 大于有序集合最大索引，Redis会将其视为有序集合的最后一个元素。

```
rdis> ZADD myzset 1 "one"
(integer) 1
rdis> ZADD myzset 2 "two"
(integer) 1
rdis> ZADD myzset 3 "three"
(integer) 1
rdis> ZRANGE myzset 0 -1
1) "one"
2) "two"
3) "three"
rdis> ZRANGE myzset 2 3
1) "three"
redis> ZRANGE myzset -2 -1
1) "two"
2) "three"
reids> ZRANGE myzset 0 1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
```
#### ZRANGEBYLEX key min max [LIMIT offset count] 
    返回指定成员区间内的成员。按成员字典正序排序, 每个成员的分数必须相同，否则返回结果不准。
**Options：**
    * min:  字典中排序位置较小的成员,必须以"[“ (包含 min)开头,或者以"(" (不包含 min)开头,可使用“-”  (‘-’代表第一个元素)代替
    * max：字典中排序位置较大的成员,必须以"[“ (包含 max)”[“ (包含 max)开头,或者以”(" (不包含 max)开头,可使用”+”元素）代替 
    * LIMIT： 返回结果是否分页,指令中包含LIMIT后offset、count必须输入
    * offset: 返回结果起始位置
    * count: 返回结果数量
```
redis> zadd zset 0 a 0 aa 0 abc 0 apple 0 b 0 c 0 d 0 d1 0 dd 0 dobble 0 z 0 z1
(integer) 12
redis> ZRANGEBYLEX zset - +
 1) "a"
 2) "aa"
 3) "abc"
 4) "apple"
 5) "b"
 6) "c"
 7) "d"
 8) "d1"
 9) "dd"
10) "dobble"
11) "z"
12) "z1"
redis> ZRANGEBYLEX zset - + LIMIT 0 3
1) "a"
2) "aa"
3) "abc"
redis> ZRANGEBYLEX zset [aa [c 
1) "aa"
2) "abc"
3) "apple"
4) "b"
5) "c"
```
**使用场景**
##### 1. 电话号码排序
```
redis> zadd phone 0 13100111100 0 13110114300 0 13132110901 
(integer) 3
redis> zadd phone 0 13200111100 0 13210414300 0 13252110901 
(integer) 3
redis> zadd phone 0 13300111100 0 13310414300 0 13352110901 
(integer) 3
// 获取132号段:
redis> ZRANGEBYLEX phone [132 (133
1) "13200111100"
2) "13210414300"
3) "13252110901"
```
##### 2. 姓名排序
```
redis> zadd names 0 Toumas 0 Jake 0 Bluetuo 0 Gaodeng 0 Aimini 0 Aidehua 
(integer) 6
// 获取名字中大写字母A开头的所有人
redis> ZRANGEBYLEX names [A (B
1) "Aidehua"
2) "Aimini"
```
#### ZLEXCOUNT key min max
    ZLEXCOUNT 命令用于计算有序集合中指定成员之间的成员数量。min 和 max 参数的含义参考 zrangebylex 命令。
    与 zrangebylex 命令的区别是：zrangebylex 返回有序几个符合条件的元素列表，而ZLEXCOUNT 返回有序集合中成员名称 min 和 max 之间的成员数量（Integer类型）。
```
redis> ZADD myzset 1 a 2 b 3 c 4 d 5 e 6 f 7 g
(integer) 7
redis> zrange myzset 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
7) "g"
redis> ZLEXCOUNT myzset - +
(integer) 7
redis> ZLEXCOUNT myzset [c +
(integer) 5
redis> ZLEXCOUNT myzset - [c
(integer) 3
redis> 
```
#### ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count] 
    返回key的有序集合中的分数在min和max之间的所有元素（默认包括分数等于max或者min的元素）。
**Options**
    * max: 最大分数值,可使用"+inf"代替
    * min: 最小分数值,可使用"-inf"代替
    * WITHSCORES: 将成员分数一并返回
    * LIMIT: 返回结果是否分页,指令中包含LIMIT后offset、count必须输入
    * offset: 返回结果起始位置
    * count: 返回结果数量
**注意：**
* min 和 max 默认使用闭区间，可以通过给 min 和 max 前面添加(符号使之变成开区间。比如 ZRANGEBYSCORE zset (1 5 返回所有符合条件1 < score <= 5的成员。
* 可选参数WITHSCORES会返回元素和其分数，而不只是元素。这个选项在redis2.0之后的版本都可用
* 如果像获取所有元素，min 和 max可以使用-inf +inf 代表最小值和最大值。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZRANGEBYSCORE myzset -inf +inf
1) "one"
2) "two"
3) "three"
redis> ZRANGEBYSCORE myzset 1 2
1) "one"
2) "two"
redis> ZRANGEBYSCORE myzset (1 2
1) "two"
redis> ZRANGEBYSCORE myzset (1 (2
(empty list or set)
redis> 
```
#### ZCOUNT key min max
    返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员。参数min和max的详细使用方法，请参考ZRANGEBYSCORE命令。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZCOUNT myzset -inf +inf
(integer) 3
redis> ZCOUNT myzset (1 3
(integer) 2
```
#### ZPOPMAX key [count] 
    删除并返回有序集合key中的最多count个具有最高分数的成员（并按分数从大到小排序），如果不提供 count，默认为 1.
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZPOPMAX myzset
1) "3"
2) "three"
redis> 
```
#### ZPOPMIN key [count] 
    删除并返回有序集合key中的最多count个具有最低得分的成员（并按分数从小到大排序），如果不提供 count，默认为 1.
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZPOPMIN myzset
1) "1"
2) "one"
redis>
```
#### ZREM key member [member ...] 
    移除一个或多个 member。返回的是从有序集合中删除的成员个数，不包括不存在的成员。
    当key存在，但是其不是有序集合类型，就返回一个错误。
**注意：**
    在2.4之前的版本中，每次只能删除一个成员。在2.4之后，接受多个参数。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREM myzset "two"
(integer) 1
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "three"
4) "3"
redis> 
```
#### ZREMRANGEBYLEX key min max 
    删除名称按字典由低到高排序成员之间所有成员。返回删除元素的个数。
**Options：**
    * min:  字典中排序位置较小的成员,必须以"[“ (包含 min)开头,或者以"(" (不包含 min)开头,可使用“-”  (‘-’代表第一个元素)代替
    * max：字典中排序位置较大的成员,必须以"[“ (包含 max)”[“ (包含 max)开头,或者以”(" (不包含 max)开头,可使用”+”元素）代替 
**注意：**
    有序集合中分数必须相同! 如果有序集合中的成员分数有不一致的,结果就不准。
```
redis> zadd zset 0 a 0 aa 0 abc 0 apple 0 b 0 c 0 d 0 d1 0 dd 0 dobble 0 z 0 z1
(integer) 12
redis> ZRANGEBYLEX zset - +
 1) "a"
 2) "aa"
 3) "abc"
 4) "apple"
 5) "b"
 6) "c"
 7) "d"
 8) "d1"
 9) "dd"
10) "dobble"
11) "z"
12) "z1"
redis> ZREMRANGEBYLEX zset [d1 (dd
(integer) 1
redis> ZRANGEBYLEX zset - +
 1) "a"
 2) "aa"
 3) "abc"
 4) "apple"
 5) "b"
 6) "c"
 7) "d"
 8) "dd"
 9) "dobble"
10) "z"
11) "z1"
```
#### ZREMRANGEBYRANK key start stop
    移除指定排名(rank)区间内的所有成员。下标参数start和stop都以0为底，0处是分数最小的那个元素。这些索引也可是负数，表示位移从最高分处开始数。例如，-1是分数最高的元素，-2是分数第二高的，依次类推。
    返回被移除成员的数量。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREMRANGEBYRANK myzset 0 1
(integer) 2
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "three"
2) "3"
redis> 
```
#### ZREMRANGEBYSCORE key min max
    移除有序集key中，所有score值介于min和max之间(包括等于min或max)的成员。 自版本2.1.6开始，score值等于min或max的成员也可以不包括在内，语法请参见ZRANGEBYSCORE命令。
    返回删除的元素的个数。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREMRANGEBYSCORE myzset -inf (2
(integer) 1
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "three"
4) "3"
redis> 
```
#### ZREVRANGE key start stop [WITHSCORES] 
    返回有序集key中，指定区间内的成员。其中成员的位置按score值递减(从大到小)来排列。具有相同score值的成员按字典序的反序排列。 除了成员按score值递减的次序排列这一点外，ZREVRANGE命令的其他方面和ZRANGE命令一样。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREVRANGE myzset 0 -1
1) "three"
2) "two"
3) "one"
redis> ZREVRANGE myzset 2 3
1) "one"
redis> ZREVRANGE myzset -2 -1
1) "two"
2) "one"
redis> 
```
#### ZREVRANGEBYLEX key max min [LIMIT offset count] 
    返回指定成员区间内的成员列表，按成员字典倒序排序, 分数必须相同。
**Options**
参数参考 ZRANGEBYLEX 命令。
```
redis> zadd zset 0 a 0 aa 0 abc 0 apple 0 b 0 c 0 d 0 d1 0 dd 0 dobble 0 z 0 z1
(integer) 12
redis> ZREVRANGEBYLEX zset + -
 1) "z1"
 2) "z"
 3) "dobble"
 4) "dd"
 5) "d1"
 6) "d"
 7) "c"
 8) "b"
 9) "apple"
10) "abc"
11) "aa"
12) "a"
// 获取分页数据
redis> ZREVRANGEBYLEX zset + - LIMIT 0 3
1) "z1"
2) "z"
3) "dobble"
// 使用 “(“ 小于号获取成员之间的元素
redis> ZREVRANGEBYLEX zset (c [aa
1) "b"
2) "apple"
3) "abc"
4) "aa"
```
#### ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count] 
    返回有序集合中指定分数区间内的成员，分数由高到低排序。
**Options**
参数参考 ZRANGE 命令。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREVRANGEBYSCORE myzset +inf -inf
1) "three"
2) "two"
3) "one"
redis> ZREVRANGEBYSCORE myzset 2 1
1) "two"
2) "one"
redis> ZREVRANGEBYSCORE myzset 2 (1
1) "two"
redis> ZREVRANGEBYSCORE myzset (2 (1
(empty list or set)
redis> 
```
#### ZSCORE key member 
    返回有序集key中，成员member的score值，如果member元素不是有序集key的成员，或key不存在，返回nil。
```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZSCORE myzset "one"
"1"
```


















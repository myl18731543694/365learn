## Redis String类型命令

#### 1. SET 命令 （SET key value [EX seconds] [PX milliseconds] [NX|XX]）

- `EX second` ：设置键的过期时间为 `second` 秒。 `SET key value EX second` 效果等同于 `SETEX key second value` 。

- `PX millisecond` ：设置键的过期时间为 `millisecond` 毫秒。 `SET key value PX millisecond` 效果等同于 `PSETEX key millisecond value` 。

- `NX` ：只在键不存在时，才对键进行设置操作。 `SET key value NX` 效果等同于 `SETNX key value` 。
- `XX` ：只在键已经存在时，才对键进行设置操作。

##### 1.1 set a 1  

如果之前存在过key，则会直接覆盖 value

> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> get a
> "1"
> 127.0.0.1:6379> set a 2
> OK
> 127.0.0.1:6379> get a
> "2"

##### 1.2 set a 1 ex 10 （ex 0会报错）

添加一个只存活10秒的 key：a

> 127.0.0.1:6379> set a 1 ex 10
> OK
> 127.0.0.1:6379> get a
> "1"
>
> 10秒后
>
> 127.0.0.1:6379> get a
> (nil)
>
> 127.0.0.1:6379> set a 1 ex 0
> (error) ERR invalid expire time in set

##### 1.3 set a 1 px 10000 （px 0会报错）

添加一个只存活10000毫秒（10秒）的 key：a

> 127.0.0.1:6379> set a 1 px 10000
> OK
> 127.0.0.1:6379> get a
> "1"
>
> 10秒后
>
> 127.0.0.1:6379> get a
> (nil)
>
> 127.0.0.1:6379> set a 1 px 0
> (error) ERR invalid expire time in set

##### 1.4 set a 1 nx（和xx相反）

当key：a 不存在时，才会添加，如果有则会忽略

> 127.0.0.1:6379> keys *
> (empty list or set)
> 127.0.0.1:6379> set a 1 nx
> OK
> 127.0.0.1:6379> get a
> "1"
> 127.0.0.1:6379> set a 2 nx
> (nil)
> 127.0.0.1:6379> get a
> "1"

##### 1.5 set a 1 xx （和nx相反）

当key：a 存在时，才会修改，否则则忽略

> 127.0.0.1:6379> keys *
> (empty list or set)
> 127.0.0.1:6379> set a 1 xx
> (nil)
> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> set a 2
> OK
> 127.0.0.1:6379> set a 1 xx
> OK
> 127.0.0.1:6379> get a
> "1"

##### 1.6 set a 1 ex 10 nx 或者 set a 1 px 10000 nx

当key：a 不存在时添加一个定时的 value：1。

> 127.0.0.1:6379> keys *
> (empty list or set)
> 127.0.0.1:6379> set a 1 ex 10 nx
> OK
> 127.0.0.1:6379> get a
> "1"
> 10秒后
> 127.0.0.1:6379> get a
> (nil)

##### 1.7 set a 1 ex 10 xx 或者 set a 1 px 10000 xx

当key：a存在时才做修改

> 127.0.0.1:6379> keys *
> (empty list or set)
> 127.0.0.1:6379> set a 1 ex 10 xx
> (nil)
> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> set a 2 ex 10 xx
> OK
> 127.0.0.1:6379> get a
> "2"
> 127.0.0.1:6379> ttl a
> (integer) 6
> 127.0.0.1:6379> get a
> (nil)

##### 1.8 java代码（总结）

> ```java
> package com.learn.learn_redis;
> 
> import java.util.concurrent.TimeUnit;
> 
> import redis.clients.jedis.Jedis;
> import redis.clients.jedis.params.SetParams;
> 
> public class TestApi {
> 	public static void main(String[] args) throws InterruptedException {
> 		Jedis jedis = new Jedis("127.0.0.1", 6379);
> 		// 清空本地库
> 		jedis.flushDB();
> 		System.out.println("--------------1.1------------------");
> 
> 		// set a 1
> 		jedis.set("a", "1");
> 		System.out.println(jedis.get("a"));
> 		jedis.set("a", "2");
> 		System.out.println(jedis.get("a"));
> 
> 		jedis.flushDB();
> 		System.out.println("--------------1.2------------------");
> 
> 		// set a 1 ex 10 （ex 0会报错）
> 		jedis.set("a", "1", SetParams.setParams().ex(1));
> 		System.out.println(jedis.get("a"));
> 		TimeUnit.SECONDS.sleep(2);
> 		System.out.println(jedis.get("a"));
> 
> 		jedis.flushDB();
> 		System.out.println("--------------1.3------------------");
> 
> 		// set a 1 px 10 （px 0会报错）
> 		jedis.set("a", "1", SetParams.setParams().px(1000));
> 		System.out.println(jedis.get("a"));
> 		TimeUnit.SECONDS.sleep(2);
> 		System.out.println(jedis.get("a"));
> 		
> 		jedis.flushDB();
> 		System.out.println("--------------1.4------------------");
> 		
> 		// set a 1 nx（和xx相反）
> 		jedis.set("a", "1", SetParams.setParams().nx());
> 		System.out.println(jedis.get("a"));
> 		jedis.set("a", "2", SetParams.setParams().nx());
> 		System.out.println(jedis.get("a"));
> 		
> 		jedis.flushDB();
> 		System.out.println("--------------1.5------------------");
> 		
> 		// set a 1 xx （和nx相反）
> 		jedis.set("a", "1", SetParams.setParams().xx());
> 		System.out.println(jedis.get("a"));
> 		jedis.set("a", "1");
> 		System.out.println(jedis.get("a"));
> 		jedis.set("a", "2", SetParams.setParams().xx());
> 		System.out.println(jedis.get("a"));
> 
> 		jedis.flushDB();
> 		System.out.println("--------------1.6------------------");
> 		
> 		// set a 1 ex 10 nx 或者 set a 1 px 10000 nx
> 		jedis.set("a", "1", SetParams.setParams().ex(1).nx());
> 		System.out.println(jedis.get("a"));
> 		TimeUnit.SECONDS.sleep(2);
> 		System.out.println(jedis.get("a"));
> 		
> 		jedis.flushDB();
> 		System.out.println("--------------1.7------------------");
> 		
> 		// set a 1 ex 10 xx 或者 set a 1 px 10000 xx
> 		jedis.set("a", "2");
> 		System.out.println(jedis.get("a"));
> 		jedis.set("a", "1", SetParams.setParams().ex(1).xx());
> 		System.out.println(jedis.get("a"));
> 		TimeUnit.SECONDS.sleep(2);
> 		System.out.println(jedis.get("a"));
> 		
> 		jedis.close();
> 	}
> 
> }
> ```
>
> ```java
> // 和ex px有关设置成0，会抛出异常
> jedis.set("a", "1", SetParams.setParams().ex(0));
> 
> Exception in thread "main" redis.clients.jedis.exceptions.JedisDataException: ERR invalid expire time in set
> 	at redis.clients.jedis.Protocol.processError(Protocol.java:132)
> 	at redis.clients.jedis.Protocol.process(Protocol.java:166)
> 	at redis.clients.jedis.Protocol.read(Protocol.java:220)
> 	at redis.clients.jedis.Connection.readProtocolWithCheckingBroken(Connection.java:318)
> 	at redis.clients.jedis.Connection.getStatusCodeReply(Connection.java:236)
> 	at redis.clients.jedis.Jedis.set(Jedis.java:166)
> 	at com.learn.learn_redis.TestApi.main(TestApi.java:25)
> ```



#### 2. **GET key**

- 返回 `key` 所关联的字符串值。
- 如果 `key` 不存在那么返回特殊值 `nil` 。
- 假如 `key` 储存的值不是字符串类型，返回一个错误，因为 GET 只能用于处理字符串值。

> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> get a
> "1"
>
> 127.0.0.1:6379> LPUSH b 1 2 3
> (integer) 3
>
> 获取的key不是存储string类型的，则报错
>
> 127.0.0.1:6379> get b
> (error) WRONGTYPE Operation against a key holding the wrong kind of value
> 127.0.0.1:6379>       



#### 3. **SETEX key seconds value**（秒级，设置带过期秒数的key，设置0秒会报错）

- 将值 `value` 关联到 `key` ，并将 `key` 的生存时间设为 `seconds` (以秒为单位)。

- 如果 `key` 已经存在， `SETEX`命令将覆写旧值。

- 这个命令类似于以下两个命令：

  ```
  SET key value ex seconds
  EXPIRE key seconds
  ```

- 不同之处是， `SETEX` 是一个原子性(atomic)操作，关联值和设置生存时间两个动作会在同一时间内完成，该命令在 Redis 用作缓存时，非常实用。

> 127.0.0.1:6379> keys *
> (empty list or set)
> 127.0.0.1:6379> setex a 10 1
> OK
> 127.0.0.1:6379> get a
> "1"
> 10秒后
> 127.0.0.1:6379> get a
> (nil)
>
> 设置0秒也会报错
>
> 127.0.0.1:6379> setex a 0 1
> (error) ERR invalid expire time in setex



#### 4. **PSETEX key milliseconds value** （毫秒级）

这个命令和 SETEX 命令相似，但它以毫秒为单位设置 `key` 的生存时间，而不是像 SETEX 命令那样，以秒为单位。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> PSETEX a 2000 1
> OK
> 127.0.0.1:6379> get a
> "1"
>
> **两秒后**
>
> 127.0.0.1:6379> get a
> (nil)



#### 5. **SETNX key value** （不存在才会设置value）

- 将 `key` 的值设为 `value` ，当且仅当 `key` 不存在。

- 若给定的 `key` 已经存在，则 `SETNX` 不做任何动作。

- `SETNX` 是『SET if Not eXists』(如果不存在，则 SET)的简写。

- 这个命令类似：

  ```
  set key value NX
  ```

> 127.0.0.1:6379> keys *
> (empty list or set)
> 127.0.0.1:6379> setnx a 1
> (integer) 1
> 127.0.0.1:6379> get a
> "1"
> 127.0.0.1:6379> setnx a 2
> (integer) 0
> 127.0.0.1:6379> get a
> "1"



#### 6. **SETBIT key offset value** （修改偏移量上的值）

在redis中，存储的字符串都是以二级制的进行存在的。

举例：

设置一个　key-value ，键的名字叫“a” 值为字符'a'

我们知道 'a' 的ASCII码是  97。转换为二进制是：01100001。offset的学名叫做“偏移” 。二进制中的每一位就是offset值啦，比如在这里  offset 0 等于 ‘0’ ，offset 1等于'1' ,offset2等于'1',offset 7 等于'1' ，没错，offset是从左往右计数的，也就是从高位往低位。



通过SETBIT 命令将 andy中的 'a' 变成 'b' 应该怎么变

也就是将 01100001 变成 01100010 （b的ASCII码是98），也就是将'a'中的offset 6从0变成1，将offset 7 从1变成0 。

每次SETBIT完毕之后，有一个（integer） 0或者（integer）1的返回值，这个是在你进行SETBIT 之前，该offset位的比特值。

> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> SETBIT a 6 1
> (integer) 0
> 127.0.0.1:6379> SETBIT a 7 0
> (integer) 1
> 127.0.0.1:6379> get a
> "b"



#### 7. **GETBIT key offset **(获取偏移量的值)

- 对 `key` 所储存的字符串值，获取指定偏移量上的位(bit)。
- 当 `offset` 比字符串值的长度大，或者 `key` 不存在时，返回 `0` 

举例：

设置一个　key-value ，键的名字叫“a” 值为字符'a'

在 `5 SETBIT key offset value` 中知道了  'a' 的ASCII码是  97。转换为二进制是：01100001

> 127.0.0.1:6379> getbit a 6
> (integer) 0
> 127.0.0.1:6379> getbit a 7
> (integer) 1



#### 8. **SETRANGE key offset value** （设置字符串区间值）

- 用 `value` 参数覆写(overwrite)给定 `key` 所储存的字符串值，从偏移量 `offset` 开始。
- 不存在的 `key` 当作空白字符串处理。
- `SETRANGE` 命令会确保字符串足够长以便将 `value` 设置在指定的偏移量上，如果给定 `key` 原来储存的字符串长度比偏移量小(比如字符串只有 `5` 个字符长，但你设置的 `offset` 是 `10` )，那么原字符和偏移量之间的空白将用零字节(zerobytes, `"\x00"` )来填充。
- 注意你能使用的最大偏移量是 2^29-1(536870911) ，因为 Redis 字符串的大小被限制在 512 兆(megabytes)以内。如果你需要使用比这更大的空间，你可以使用多个 `key` 。

> 127.0.0.1:6379> set a 12345
> OK
>
> 从下标2开始 也就是 3 开始替换 34567
>
> 127.0.0.1:6379> SETRANGE a 2 34567
> (integer) 7
>
> 127.0.0.1:6379> get a
> "1234567"
>
> 127.0.0.1:6379> SETRANGE a 2 22
> (integer) 7
> 127.0.0.1:6379> get a
> "1222567"
>
> 如果offset不够用 \x00（占一位） 添加
>
> 127.0.0.1:6379> SETRANGE a 10 2
> (integer) 11
>
> 127.0.0.1:6379> get a
> "1222567\x00\x00\x002"
>
> 指定offset为负数则报错
>
> 127.0.0.1:6379> SETRANGE a -1 111
> (error) ERR offset is out of range



#### 9. **GETRANGE key start end **（获取字符串区间的值）

- 返回 `key` 中字符串值的子字符串，字符串的截取范围由 `start` 和 `end` 两个偏移量决定(包括 `start` 和 `end` 在内)。
- 负数偏移量表示从字符串最后开始计数， `-1` 表示最后一个字符， `-2` 表示倒数第二个，以此类推。
- `GETRANGE` 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。
- 这个方法很像java里的 String substring(int beginIndex, int endIndex) ，切割字符串



> 包括了 1 和 2
> 127.0.0.1:6379> set a 12345
> OK
> 127.0.0.1:6379> GETRANGE a 1 2
> "23"
>
> 如果范围 end 和 start区间有问题会返回 空字符串
> 127.0.0.1:6379> set a 12345
> OK
> 127.0.0.1:6379> GETRANGE a 2 1
> ""
> 127.0.0.1:6379> GETRANGE a 4 -2
> ""



#### 10. **GETSET key value** （设置并返回之前值）

- 将给定 `key` 的值设为 `value` ，并返回 `key` 的旧值(old value)。
- 当 `key` 存在但不是字符串类型时，返回一个错误。



> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> GETSET a 2
> "1"
> 127.0.0.1:6379> get a
> "2"
>
> **如果没有旧值，返回null**
>
> 127.0.0.1:6379> getset b 1
> (nil)



#### 11. **STRLEN key** （返回key对应字符串的长度）

- 返回 `key` 所储存的字符串值的长度。
- 当 `key` 储存的不是字符串值时，返回一个错误。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a 123456
> OK
> 127.0.0.1:6379> STRLEN a
> (integer) 6
>
> **不存在的value，返回0**
>
> 127.0.0.1:6379> STRLEN b
> (integer) 0
>
> **不是string类型则报错**
>
> 127.0.0.1:6379> lpush c 1 2 3
> (integer) 3
> 127.0.0.1:6379> STRLEN c
> (error) WRONGTYPE Operation against a key holding the wrong kind of value



#### 12. **MSET key value [key value ...]** (批量设置key value)

- 同时设置一个或多个 `key-value` 对。
- 如果某个给定 `key` 已经存在，那么 MSET 会用新值覆盖原来的旧值，如果这不是你所希望的效果，请考虑使用 MSETNX 命令：它只会在所有给定 `key` 都不存在的情况下进行设置操作。
- MSET 是一个原子性(atomic)操作，所有给定 `key` 都会在同一时间内被设置，某些给定 `key` 被更新而另一些给定 `key` 没有改变的情况，不可能发生。

> 127.0.0.1:6379> FLUSHDB
> OK
> 127.0.0.1:6379> mset a a b b c c
> OK
> 127.0.0.1:6379> mget a b c
> 1) "a"
> 2) "b"
> 3) "c"
>
> **如果有部分key存在，则会直接覆盖**
>
> 127.0.0.1:6379> mset c d e e
> OK
> 127.0.0.1:6379> get c
> "d"
> **如果某个key的value不是string，也会直接覆盖**



#### 13. **MGET key [key ...]** （批量获取）

- 返回所有(一个或多个)给定 `key` 的值。
- 如果给定的 `key` 里面，有某个 `key` 不存在，那么这个 `key` 返回特殊值 `nil` 。因此，该命令永不失败。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> set b b
> OK
> 127.0.0.1:6379> lpush c 1 2 3
> (integer) 3
>
> **如果中间某个不为string类型，获取则是null**
>
> **不存在的key也是返回null**
>
> 127.0.0.1:6379> mget a b c d
> 1) "1"
> 2) "b"
> 3) (nil)
> 4) (nil)



#### 14. **MSETNX key value [key value ...]**

- 同时设置一个或多个 `key-value` 对，当且仅当所有给定 `key` 都不存在。
- 即使只有一个给定 `key` 已存在，MSETNX 也会拒绝执行所有给定 `key` 的设置操作。
- MSETNX 是原子性的，因此它可以用作设置多个不同 `key` 表示不同字段(field)的唯一性逻辑对象(unique logic object)，所有字段要么全被设置，要么全不被设置。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a a
> OK
>
> **key a存在，则本次都取消**
>
> 127.0.0.1:6379> MSETNX a 1 b b c c
> (integer) 0
> 127.0.0.1:6379> get a
> "a"
> 127.0.0.1:6379> get b
> (nil)
>
> **只有所有key不存在，才会添加**
>
> 127.0.0.1:6379> msetnx b b c c
> (integer) 1
> 127.0.0.1:6379> get b
> "b"



#### 15. **APPEND key value**

- 如果 `key` 已经存在并且是一个字符串， APPEND 命令将 `value` 追加到 `key` 原来的值的末尾。
- 如果 `key` 不存在，APPEND 就简单地将给定 `key` 设为 `value` ，就像执行 `SET key value` 一样。

> 127.0.0.1:6379> flushdb
> OK
>
> **没有相当于 set key value**
>
> 127.0.0.1:6379> APPEND a 12
> (integer) 2
> 127.0.0.1:6379> get a
> "12"
>
> **有key则就追加字符**
>
> 127.0.0.1:6379> APPEND a 34
> (integer) 4
> 127.0.0.1:6379> get a
> "1234"
>
> **如果类型不是string，则会报错**
>
> 127.0.0.1:6379> lpush b 1 2 4 5
> (integer) 4
> 127.0.0.1:6379> APPEND b 67
> (error) WRONGTYPE Operation against a key holding the wrong kind of value



#### 16. **DECR key** （字符减一） 和 **DECRBY key decrement** （字符减 decrement）

**DECR key**

- 将 `key` 中储存的数字值减一。
- 如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 [DECR](http://doc.redisfans.com/string/decr.html#decr) 操作。
- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
- 本操作的值限制在 64 位(bit)有符号数字表示之内。
- 关于递增(increment) / 递减(decrement)操作的更多信息，请参见 [*INCR*](http://doc.redisfans.com/string/incr.html#incr) 命令。

**DECRBY key decrement**

- 将 `key` 所储存的值减去减量 `decrement` 。
- 如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 [DECRBY](http://doc.redisfans.com/string/decrby.html#decrby) 操作。
- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
- 本操作的值限制在 64 位(bit)有符号数字表示之内。
- 关于更多递增(increment) / 递减(decrement)操作的更多信息，请参见 [*INCR*](http://doc.redisfans.com/string/incr.html#incr) 命令。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a 1
> OK
>
> **减一变成0**
>
> 127.0.0.1:6379> DECR a
> (integer) 0
> 127.0.0.1:6379> get a
> "0"
>
> **不是int 报错**
>
> 127.0.0.1:6379> set b a
> OK
> 127.0.0.1:6379> DECR b
> (error) ERR value is not an integer or out of range
>
> **指定减3**
>
> 127.0.0.1:6379> DECRBY a 3
> (integer) -3
> 127.0.0.1:6379> get a
> "-3"



#### 17. **INCR key** （自增1） 和 **INCRBY key increment **（自增 increment）

**INCR key**

- 将 `key` 中储存的数字值增一。
- 如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 [INCR](http://doc.redisfans.com/string/incr.html#incr) 操作。
- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
- 本操作的值限制在 64 位(bit)有符号数字表示之内。

**INCRBY key increment**

- 将 `key` 所储存的值加上增量 `increment` 。
- 如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 [INCRBY](http://doc.redisfans.com/string/incrby.html#incrby) 命令。
- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
- 本操作的值限制在 64 位(bit)有符号数字表示之内。
- 关于递增(increment) / 递减(decrement)操作的更多信息，参见 [*INCR*](http://doc.redisfans.com/string/incr.html#incr) 命令。

> 127.0.0.1:6379> flushdb
> OK
>
> **INCR 自增1**
>
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> INCR a
> (integer) 2
> 127.0.0.1:6379> get a
> "2"
>
> **INCRBY  自增3**
>
> 127.0.0.1:6379> INCRBY a 3
> (integer) 5
>
> **不是int会报错**
>
> 127.0.0.1:6379> set b b
> OK
>
> 127.0.0.1:6379> INCR b
> (error) ERR value is not an integer or out of range            



#### 18. **INCRBYFLOAT key increment** （带浮点的自增）

- 为 `key` 中所储存的值加上浮点数增量 `increment` 。
- 如果 `key` 不存在，那么 [INCRBYFLOAT](http://doc.redisfans.com/string/incrbyfloat.html#incrbyfloat) 会先将 `key` 的值设为 `0` ，再执行加法操作。
- 如果命令执行成功，那么 `key` 的值会被更新为（执行加法之后的）新值，并且新值会以字符串的形式返回给调用者。
- 无论是 `key` 的值，还是增量 `increment` ，都可以使用像 `2.0e7` 、 `3e5` 、 `90e-2` 那样的指数符号(exponential notation)来表示，但是，**执行 INCRBYFLOAT 命令之后的值**总是以同样的形式储存，也即是，它们总是由一个数字，一个（可选的）小数点和一个任意位的小数部分组成（比如 `3.14` 、 `69.768` ，诸如此类)，小数部分尾随的 `0` 会被移除，如果有需要的话，还会将浮点数改为整数（比如 `3.0` 会被保存成 `3` ）。
- 除此之外，无论加法计算所得的浮点数的实际精度有多长， [INCRBYFLOAT](http://doc.redisfans.com/string/incrbyfloat.html#incrbyfloat) 的计算结果也最多只能表示小数点的后十七位。
- 当以下任意一个条件发生时，返回一个错误：
- - `key` 的值不是字符串类型(因为 Redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
  - `key` 当前的值或者给定的增量 `increment` 不能解释(parse)为双精度浮点数(double precision floating point number）

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> INCRBYFLOAT a 0.15
> "1.15"
> 127.0.0.1:6379> get a
> "1.15"



#### 19. **BITCOUNT key [start] [end]**

- 计算给定字符串中，被设置为 `1` 的比特位的数量。
- 一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 `start` 或 `end` 参数，可以让计数只在特定的位上进行。
- `start` 和 `end` 参数的设置和 [*GETRANGE*](http://doc.redisfans.com/string/getrange.html#getrange) 命令类似，都可以使用负数值：比如 `-1` 表示最后一个位，而 `-2` 表示倒数第二个位，以此类推。
- 不存在的 `key` 被当成是空字符串来处理，因此对一个不存在的 `key` 进行 `BITCOUNT` 操作，结果为 `0` 。

> **a的ASCII码是  97。转换为二进制是：01100001。bit计算1的数量为3**
>
> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> bitcount a
> (integer) 3
>
> 127.0.0.1:6379> set a aa
> OK
> 127.0.0.1:6379> BITCOUNT a
> (integer) 6
>
> **这里计算的是value字符串start开始到end结束 bit为1的数量**
>
> 127.0.0.1:6379> set a abc
> OK
> 127.0.0.1:6379> BITCOUNT a 0 0
> (integer) 3
> 127.0.0.1:6379> BITCOUNT a 1 1
> (integer) 3
> 127.0.0.1:6379> BITCOUNT a 2 2
> (integer) 4



#### 20. **BITOP operation destkey key [key ...]** (2020年5月10日 看不懂，暂时保留)

- 对一个或多个保存二进制位的字符串 `key` 进行位元操作，并将结果保存到 `destkey` 上。
- `operation` 可以是 `AND` 、 `OR` 、 `NOT` 、 `XOR` 这四种操作中的任意一种：
- - `BITOP AND destkey key [key ...]` ，对一个或多个 `key` 求逻辑并，并将结果保存到 `destkey` 。
  - `BITOP OR destkey key [key ...]` ，对一个或多个 `key` 求逻辑或，并将结果保存到 `destkey` 。
  - `BITOP XOR destkey key [key ...]` ，对一个或多个 `key` 求逻辑异或，并将结果保存到 `destkey` 。
  - `BITOP NOT destkey key` ，对给定 `key` 求逻辑非，并将结果保存到 `destkey` 。
- 除了 `NOT` 操作之外，其他操作都可以接受一个或多个 `key` 作为输入。
- **处理不同长度的字符串**
- 当 [BITOP](http://doc.redisfans.com/string/bitop.html#bitop) 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 `0` 。
- 空的 `key` 也被看作是包含 `0` 的字符串序列。



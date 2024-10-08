
## 简介

​	`lemon-cache`是基于`redis`封装的操作工具集，服务于业务的缓存相关管理。扩展了Spring cache的基础能力，方便业务使用，快速上手。以下内容主要由几点进行展开：

- CacheExpire：对Spring Cache的增强，通过注解方式提供了对缓存过期时间管理的支持；

- CacheHelper：对Spring RedisTemplate的操作工具集封装，对Redis五种基本类型的各种操作；

## 安装

1. 在项目的`pom.xml`中添加`parent`父依赖

```xml
<parent>
    <groupId>com.lemon.framework</groupId>
	<artifactId>lemon-dependencies</artifactId>
	<version>1.0.0.RELEASE</version>
</parent>
```

2. 在项目`pom.xml`的`dependencies`中加入以下内容：

```xml
<dependency>
    <groupId>com.lemon.framework</groupId>
  <artifactId>lemon-cache-spring-boot-starter</artifactId>
</dependency>
```

## 扩展配置

> 可在`application.yml`根据自身情况灵活配置以下属性

```yml
lemon:
# 缓存管理
cache:
  manager:
    enable: true # 缓存管理器配置开关, 默认值: true
    default-expire-type: hours # 缓存管理器默认过期时间类型, 可选值: SECONDS/MINUTES/HOURS/DAY/WEEK/MONTH/YEAR, 默认值: hours
    default-expire-time: 1 # 缓存管理器默认过期时间, 默认为1小时
```

## CacheExpire

### 简介

​	`Spring Cache`的注解方式，`@Cacheable`, `@CachePut`和 `@CacheEvict`组合用起来真的很方便，比直接使用RedisTemplate的接口简洁太多。但是`@Cacheable`不支持指定过期时间(TTL)，只能通过申明`CacheManager bean`指定case的过期策略，使用起来极为不灵活。介于综合以上所述，框架提供了`@CacheExpire`注解动态指定过期策略，方便统一维护更新。

### 如何使用

#### 扩展配置

```yml
  lemon:
    # 缓存管理 
    cache:
      manager:
      	enable: true # 缓存管理器配置开关, 默认值: true
        default-expire-type: hours # 缓存管理器默认过期时间类型, 可选值: SECONDS/MINUTES/HOURS/DAY/WEEK/MONTH/YEAR, 默认值: hours
        default-expire-time: 1 # 缓存管理器默认过期时间, 默认为1小时
```

#### 注解说明

> `@CacheExpire`注解类如下：

```java
/**
 * 缓存过期配置
 *
 * @author siven
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheExpire {
    /**
     * expire time, default 60s
     * 过期时间, 默认60
     */
    @AliasFor("expire")
    long value() default 60L;

    /**
     * expire time, default 60s
     * 过期时间, 默认60
     */
    @AliasFor("value")
    long expire() default 60L;

    /**
     * expire time type , default SECONDS
     * 过期时间类型, 默认秒
     *
     * @return
     */
    ExpireTimeType type() default ExpireTimeType.SECONDS;
}
```

#### 使用示例

```java
import com.lemon.framework.cache.CacheExpire;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.CacheConfig;

@CacheConfig(cacheNames = "cache")
public class CacheTest {
  
  	....
  
    /**
     * 缓存的KEY为user:#userNo
     * 过期策略为1分钟
     * @return
     */
  	@Cacheable(key = "#userNo")
    @CacheExpire(expire = 1, type = ExpireTimeType.MINUTES)
    public User getByUserNo(@PathVariable String userNo) {
      	User user = userService.getByUserNo(userNo);
      	return user;
    }
  
    /**
     * 缓存的KEY为user:#userNo
     * 不添加@CacheExpire注解, 则采用属性配置的默认过期策略, 可以通过yml调整默认配置
     * @return
     */
		@Cacheable(value = "getByUserNo2",key = "#userNo")
    public User getByUserNo2(@PathVariable String userNo) {
      	User user = userService.getByUserNo(userNo);
      	return user;
    }
}
```

#### 代码示例

详细代码示例参看 [[quickstart-Sample](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples/-/tree/master/lemon-sample-quickstart)]

### 知识补充

- [Spring Boot缓存注解@Cacheable、@CacheEvict、@CachePut使用](https://blog.csdn.net/dreamhai/article/details/80642010)

## CacheHelper

### 简介

​	最全的Java操作Redis的工具类，封装了对Redis五种基本类型的各种操作，力求符合Redis的原生操作，使用StringRedisTemplate实现！

**快速导航：**<br>
[key命令操作](#一、keys相关命令) | 
[String数据操作](#二、String数据类型操作) | 
[Hash数据操作](#三、hash相关的操作) | 
[List数据操作](#四、list相关的操作) | 
[Set数据操作](#五、set相关的操作) | 
[zSet数据操作](#六、zset数据类型操作)

**相关知识：**<br>
[Redis基础知识补充](#知识补充) | 
[集成使用](#三、集成使用) | 

### 解惑

&emsp;很多人提出疑问“为什么没有操作Object的方法？”，请看这里介绍[redistemplate和stringredistemplate](#二、RedisTemplate和StringRedisTemplate)。 

### API

#### 一、keys相关命令

|  NO  | 方法                                                    |  描述                                    |
|:----:|---------------------------------------------------------|-----------------------------------------|
|   1  | void delete(String key)                                 | key存在时删除key                         |
|   2  | void delete(Collection keys)                            | 批量删除key                              |
|   3  | byte[] dump(String key)                                 | 序列化key，返回被序列化的值               |
|   4  | Boolean hasKey(String key)                              | 检查key是否存在                          |
|   5  | Boolean expire(String key, long timeout, TimeUnit unit) | 设置过期时间                             |
|   6  | Boolean expireAt(String key, Date date)                 | 设置过期时间                             |
|   7  | Set<String> keys(String pattern)                        | 查找所有符合给定模式(pattern)的key        |
|   8  | Boolean move(String key, int dbIndex)                   | 将当前数据库的key移动到给定的数据库db当中  |
|   9  | Boolean persist(String key)                             | 移除key的过期时间，key将持久保持          |
|  10  | Long getExpire(String key, TimeUnit unit)               | 返回key的剩余的过期时间                   |
|  11  | Long getExpire(String key)                              | 返回key的剩余的过期时间                   |
|  12  | String randomKey()                                      | 从当前数据库中随机返回一个key             |
|  13  | void rename(String oldKey, String newKey)               | 修改key的名称                            |
|  14  | Boolean renameIfAbsent(String oldKey, String newKey)    | 仅当newkey不存在时，将oldKey改名为 newkey |
|  15  | DataType type(String key)                               | 返回key所储存的值的类型                   |

> **TimeUnit是时间单位，可选值有：**<br>
> &emsp;天:TimeUnit.DAYS、小时:TimeUnit.HOURS、分钟:TimeUnit.MINUTES、秒:TimeUnit.SECONDS、毫秒:TimeUnit.MILLISECONDS。

#### 二、String数据类型操作

|  NO  | 方法                                                              | 描述                                 |
|:----:|-------------------------------------------------------------------|-------------------------------------|
|   1  | String get(String key)                                            | 获取指定key的值                      |
|   2  | String getRange(String key, long start, long end)                 | 返回key中字符串值的子字符             |
|   3  | String getAndSet(String key, String value)                        | 将key的值设为value，并返回key旧值     |
|   4  | Boolean getBit(String key, long offset)                           | 对key所储存的值，获取指定位置上的bit   |
|   5  | List multiGet(Collection keys)                                    | 批量获取                             |
|      |     **添加相关**                                                 |                                      |
|   6  | void set(String key, String value)                                | 设置指定key的值                       |
|   7  | boolean setBit(String key, long offset, boolean value)            | 设置指定位置上的ASCII码               |
|   8  | void setEx(String key,String value,long timeout,TimeUnit unit)    | 将值value关联到key，并设置key过期时间  |
|   9  | boolean setIfAbsent(String key, String value)                     | 只有在 key 不存在时设置 key 的值       |
|  10  | void setRange(String key, String value, long offset)              | 用value覆写key的值，从偏移量offset开始 |
|  11  | void multiSet(Map<String,String> maps)                            | 批量添加                              |
|  12  | boolean multiSetIfAbsent(Map<String,String> maps)                 | 批量添加，仅当所有key都不存在          |
|      |      **其他方法**                                                 |                                      |
|  13  | Integer append(String key, String value)                          | 追加到末尾                            |
|  14  | Long incrBy(String key, long increment)                           | 增加(自增长), 负数则为自减             |
|  15  | Double incrByFloat(String key, double increment)                  | 增加(自增长), 负数则为自减             |
|  16  | Long size(String key)                                             | 获取字符串的长度                      |

> **关于上面xxBit方法的使用：**<br>
> &emsp;例如字符'a'的ASCII码是97，转为二进制是'01100001'，setBit方法就是把第offset位置上变成0或者1，true是1，false是0。

#### 三、Hash相关的操作 

|  NO  | 方法                                                           | 描述                                   |
|:----:|----------------------------------------------------------------|---------------------------------------|
|   1  | Object hGet(String key, String field)                          | 获取存储在哈希表中指定字段的值           |
|   2  | Map hGetAll(String key)                                        | 获取所有给定字段的值                    |
|   3  | List hMultiGet(String key, Collection fields)                  | 获取所有给定字段的值                    |
|      |    **添加相关**                                               |                                       |
|   4  | void hPut(String key, String hashKey, String value)            | 添加字段                               |
|   5  | void hPutAll(String key, Map maps)                             | 添加多个字段                            |
|   6  | Boolean hPutIfAbsent(String key,String hashKey,String value)   | 仅当hashKey不存在时才设置                |
|      |     **其他方法**                                              |                                        |
|   7  | Long hDelete(String key, Object... fields)                     | 删除一个或多个哈希表字段                 |
|   8  | boolean hExists(String key, String field)                      | 查看哈希表key中指定的字段是否存在         |
|   9  | Long hIncrBy(String key, Object field, long increment)         | 为哈希表key中指定字段的值增加increment   |
|  10  | Double hIncrByFloat(String key, Object field, double delta)    | 为哈希表key中指定字段的值增加increment   |
|  11  | Set hKeys(String key)                                          | 获取所有哈希表中的字段                   |
|  12  | Long hSize(String key)                                         | 获取哈希表中字段的数量                   |
|  13  | List hValues(String key)                                       | 获取哈希表中所有值                       |
|  14  | Cursor hScan(String key, ScanOptions options)                  | 迭代哈希表中的键值对                     |

#### 四、List相关的操作

|  NO  | 方法                                                     | 描述                                        |
|:----:|----------------------------------------------------------|---------------------------------------------|
|   1  | String lIndex(String key, long index)                    | 通过索引获取列表中的元素                      |
|   2  | List lRange(String key, long start, long end)            | 获取列表指定范围内的元素                      |
|      |     **添加相关**                                        |                                             |
|   3  | Long lLeftPush(String key, String value)                 | 存储在list头部                               |
|   4  | Long lLeftPushAll(String key, String... value)           | 存储在list头部                               |
|   5  | Long lLeftPushAll(String key, Collection value)          | 存储在list头部                               |
|   6  | Long lLeftPushIfPresent(String key, String value)        | 当list存在的时候才加入                        |
|   7  | lLeftPush(String key, String pivot, String value)        | 如果pivot存在,再pivot前面添加                 |
|      |                                                          |                                             |
|   8  | Long lRightPush(String key, String value)                | 存储在list尾部                               |
|   9  | Long lRightPushAll(String key, String... value)          | 存储在list尾部                               |
|  10  | Long lRightPushAll(String key, Collection value)         | 存储在list尾部                               |
|  11  | Long lRightPushIfPresent(String key, String value)       | 当list存在的时候才加入                        |
|  12  | lRightPush(String key, String pivot, String value)       | 在pivot元素的右边添加值                       |
|      |                                                          |                                              |
|  13  | void lSet(String key, long index, String value)          | 通过索引设置列表元素的值                       |
|      |    **删除相关**                                          |                                             |
|  14  | String lLeftPop(String key)                              | 移出并获取列表的第一个元素                     |
|  15  | String lBLeftPop(String key,long timeout,TimeUnit unit)  | 移出并获取第一个元素,没有则阻塞直到超时或有为止  |
|      |                                                          |                                              |
|  16  | String lRightPop(String key)                             | 移除并获取列表最后一个元素                     |
|  17  | String lBRightPop(String key,long timeout,TimeUnit unit) | 移出并获取最后个元素,没有则阻塞直到超时或有为止  |
|  18  | String lRightPopAndLeftPush(String sKey,String dKey)     | 移除最后一个元素并加到另一个列表并返回          |
|  19  | String lBRightPopAndLeftPush(sKey,dKey,timeout,unit)     | 移除最后个元素并加到另个列表并返回,阻塞超时或有  |
|      |                                                          |                                              |
|  20  | Long lRemove(String key, long index, String value)       | 删除集合中值等于value得元素                    |
|  21  | void lTrim(String key, long start, long end)             | 裁剪list                                     |
|      |     **其他方法**                                         |                                              |
|  22  | Long lLen(String key)                                    | 获取列表长度                                  |

#### 五、Set相关的操作

|  NO  | 方法                                                                     | 描述                          |
|:----:|--------------------------------------------------------------------------|-------------------------------|
|   1  | Set<String> sMembers(String key)                                         | 获取集合所有元素               |
|   2  | Long sSize(String key)                                                   | 获取集合大小                   |
|   3  | Boolean sIsMember(String key, Object value)                              | 判断集合是否包含value          |
|   4  | String sRandomMember(String key)                                         | 随机获取集合中的一个元素       |
|   5  | List<String> sRandomMembers(String key, long count)                      | 随机获取集合count个元素        |
|   6  | Set<String> sDistinctRandomMembers(String key, long count)               | 随机获取count个元素并去除重复的 |
|   7  | Cursor<String> sScan(String key, ScanOptions options)                    | 使用迭代器获取元素             |
|      |                                                                          |                               |
|   8  | Set<String> sIntersect(String key, String otherKey)                      | 获取两个集合的交集             |
|   9  | Set<String> sIntersect(String key, Collection<String> otherKeys)         | 获取key集合与多个集合的交集     |
|  10  | Long sIntersectAndStore(String key, String oKey, String dKey)            | key集合与oKey的交集存储到dKey中 |
|  11  | Long sIntersectAndStore(String key,Collection<String> oKeys,String dKey) | key与多个集合的交集存储到dKey中 |
|      |                                                                          |                               |
|  12  | Set<String> sUnion(String key, String otherKeys)                         | 获取两个集合的并集             |
|  13  | Set<String> sUnion(String key, Collection<String> otherKeys)             | 获取key集合与多个集合的并集     |
|  14  | Long sUnionAndStore(String key, String otherKey, String destKey)         | key集合与oKey的并集存储到dKey中 |
|  15  | Long sUnionAndStore(String key,Collection<String> oKeys,String dKey)     | key与多个集合的并集存储到dKey中 |
|      |                                                                          |                               |
|  16  | Set<String> sDifference(String key, String otherKey)                     | 获取两个集合的差集             |
|  17  | Set<String> sDifference(String key, Collection<String> otherKeys)        | 获取key集合与多个集合的差集     |
|  18  | Long sDifference(String key, String otherKey, String destKey)            | key与oKey集合的差集存储到dKey中 |
|  19  | Long sDifference(String key,Collection<String> otherKeys,String dKey)    | key与多个集合的差集存储到dKey中 |
|      |    **添加相关**                                                         |                                |
|  20  | Long sAdd(String key, String... values)                                  | 添加                           |
|      |    **删除相关**                                                          |                               |
|  21  | Long sRemove(String key, Object... values)                               | 移除                           |
|  22  | String sPop(String key)                                                  | 随机移除一个元素                |
|  23  | Boolean sMove(String key, String value, String destKey)                  | 将key集合中value移到destKey中   |

#### 六、zset数据类型操作

|  NO  | 方法                                                                       | 描述                             |
|:----:|----------------------------------------------------------------------------|----------------------------------|
|   1  | Set<String> zRange(String key, long start, long end)                       | 获取元素,小到大排序,s开始e结束位置 |
|   2  | Set<TypedTuple<String>> zRangeWithScores(String key, long start, long end) | 获取集合元素, 并且把score值也获取  |
|   3  | Set<String> zRangeByScore(String key, double min, double max)              | 根据score范围查询元素,从小到大排序 |
|   4  | Set<TypedTuple<String>> zRangeByScoreWithScores(key,double min,double max) | 根据score范围查询元素,并返回score |
|   5  | Set<TypedTuple> zRangeByScoreWithScores(key,double min,max,long start,end) | 根据score查询元素,s开始e结束位置   |
|      |                                                                            |                                  |
|   6  | Set<String> zReverseRange(String key, long start, long end)                | 获取集合元素, 从大到小排序         |
|   7  | Set<TypedTuple<String>> zReverseRangeWithScores(key, long start, long end) | 获取元素,从大到小排序,并返回score  |
|   8  | Set<String> zReverseRangeByScore(String key, double min, double max)       | 根据score范围查询元素,从大到小排序 |
|   9  | Set<TypedTuple> zReverseRangeByScoreWithScores(key,double min,double max)  | 根据score查询,大到小排序返回score |
|  10  | Set<String> zReverseRangeByScore(key, double min, max, long start, end)    | 根据score查询,大到小,s开始e结束   |
|      |                                                                            |                                  |
|  11  | Long zRank(String key, Object value)                                       | 返回元素在集合的排名,score由小到大 |
|  12  | Long zReverseRank(String key, Object value)                                | 返回元素在集合的排名,score由大到小 |
|  13  | Long zCount(String key, double min, double max)                            | 根据score值范围获取集合元素的数量  |
|  14  | Long zSize(String key)                                                     | 获取集合大小                      |
|  15  | Long zZCard(String key)                                                    | 获取集合大小                      |
|  16  | Double zScore(String key, Object value)                                    | 获取集合中value元素的score值      |
|      |                                                                            |                                  |
|  17  | Long zUnionAndStore(String key, String otherKey, String destKey)           | 获取key和oKey的并集并存储在dKey中 |
|  18  | Long zUnionAndStore(String key,Collection<String> otherKeys,String dKey)   | 获取key和多个集合并集并存在dKey中  |
|      |                                                                            |                                  |
|  19  | Long zIntersectAndStore(String key, String otherKey, String destKey)       | 获取key和oKey交集并存在destKey中  |
|  20  | Long zIntersectAndStore(String key,Collection<String> oKeys,String dKey)   | 获取key和多个集合交集并存在dKey中  |
|      |                                                                            |                                  |
|  21  | Cursor<TypedTuple<String>> zScan(String key, ScanOptions options)          | 使用迭代器获取                    |
|      |    **添加相关**                                                            |                                 |
|  22  | Boolean zAdd(String key, String value, double score)                       | 添加元素,zSet按score由小到大排列  |
|  23  | Long zAdd(String key, Set<TypedTuple<String>> values)                      | 批量添加,TypedTuple使用见下面介绍 |
|      |    **删除相关**                                                           |                                  |
|  24  | Long zRemove(String key, Object... values)                                 | 移除                             |
|  25  | Double zIncrementScore(String key, String value, double delta)             | 增加元素的score值,并返回增加后的值 |
|  26  | Long zRemoveRange(String key, long start, long end)                        | 移除指定索引位置的成员            |
|  27  | Long zRemoveRangeByScore(String key, double min, double max)               | 根据指定的score值的范围来移除成员  |

> **批量添加时`TypedTuple`的使用：**<br>
> TypedTuple<String> typedTuple = new DefaultTypedTuple<String>(value,score)

### 如何使用

使用CacheHelper工具类方法如下：

```java
@Autowired
private CacheHelper cacheHelper;
```

使用`@Autowired`自动注入CacheHelper。

### 知识补充

#### 一、Redis知识补充
&emsp;Redis 可以存储键与5种不同数据结构类型之间的映射，这5种数据结构类型分别为：String（字符串）、List（列表）、Set（集合）、Hash（散列）和 zSet（有序集合）。

#####  1.String（字符串）

**结构存储的值：** 

&emsp;可以是字符串、整数或者浮点数。

**结构的读写能力：**

&emsp;对整个字符串或者字符串的其中一部分执行操作，对象和浮点数执行自增(increment)或者自减(decrement)。

##### 2. List（列表）

**结构存储的值：**

&emsp;一个链表，链表上的每个节点都包含了一个字符串。

**结构的读写能力：**

&emsp;从链表的两端推入或者弹出元素，根据偏移量(offset)对链表进行修剪(trim)，读取单个或者多个元素，根据值来查找或者移除元素。

##### 3. Set（集合）

**结构存储的值：**
&emsp;包含字符串的无序收集器(unOrderedCollection)，并且被包含的每个字符串都是独一无二的、各不相同。

**结构的读写能力：**
&emsp;添加、获取、移除单个元素，检查一个元素是否存在于某个集合中，计算交集、并集、差集，从集合里面随机获取元素。

##### 4. Hash（散列）

**结构存储的值：**
&emsp;包含键值对的无序散列表。

**结构的读写能力：**
&emsp;添加、获取、移除单个键值对，获取所有键值对。

##### 5. zSet（有序集合）

**结构存储的值：**
&emsp;字符串成员(member)与浮点数分值(score)之间的有序映射，元素的排列顺序由分值(score)的大小决定。

**结构的读写能力：**
&emsp;添加、获取、删除单个元素，根据分值(score)范围(range)或者成员来获取元素。

#### 二、RedisTemplate和StringRedisTemplate

&emsp;&emsp;二者主要区别是他们使用的序列化类不一样，RedisTemplate使用的是JdkSerializationRedisSerializer， 
StringRedisTemplate使用的是StringRedisSerializer，两者的数据是不共通的。

##### 1. RedisTemplate：
&emsp;&emsp;RedisTemplate使用的是JDK的序列化策略，向Redis存入数据会将数据先序列化成字节数组然后在存入Redis数据库，
这个时候打开Redis查看的时候，你会看到你的数据不是以可读的形式展现的，而是以字节数组显示，类似下面：`\xAC\xED\x00\x05t\x05sr\x00`。 

&emsp;&emsp;所以使用RedisTemplate可以把一个Java对象直接存储在Redis里面，但是存进去的数据是不易直观读的，不通用的，
建议不要直接存一个Object对象，可以变成Hash来存储，也可以转成json格式的数据来存储，在实际应用中也是很多都采用json格式来存储的。

##### 2. StringRedisTemplate:

&emsp;&emsp;StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的，
StringRedisTemplate是继承RedisTemplate的，这种对redis的操方式更优雅，任何Redis连接工具，都可以读出直观的数据，便于数据的维护。





## DistributeLock

### 简介

使用redisson实现分布式锁注解

```java
/***
 * @description: redis 分布式锁注解
 **/
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface DistributeLock {

    /**
     * 锁的key
     * redis key的拼写规则为 "lemon:" + lockKey + ":" + {@see KeyGenerator.generate}
     */
    String lockKey();

    /**
     * 持锁时间
     * 单位秒,默认120秒<br/>
     */
    long leaseTime() default 120;

    /**
     * 没有获取到锁时，等待时间
     * 单位秒, 默认30s
     */
    long waitTime() default 30;
}
```



### 如何使用

在方法上添加@DistributeLock注解即可

```java
@Slf4j
@RequestMapping("sequence")
@RestController
public class SequenceResource {

    @Autowired
    SequenceGenerator sequenceGenerator;

    @Autowired
    SequenceIncrementInitializer sequenceIncrementInitializer;

    @DistributeLock(lockKey = "sequence")
    @LogAnnotation
    @GetMapping
    public String sequence() throws InterruptedException {
        String generator = sequenceGenerator.generator(Sequence.TEST);
        Thread.sleep(5000);
        return generator;
    }
}
```



### 分布式锁异常处理器

获取锁超时或其他异常, 业务需要对异常进行额外处理, 可实现接口即可

```java
/**
 * 分布式锁异常处理器<br>
 * <br>
 *
 * @author siven
 * @Date 2020-12-11 11:23
 */
public interface DistributeLockExceptionProcessor {

    /**
     * support lock key
     * 注意: lock key 不允许重读
     */
    String lockKey();

    /**
     * 处理异常
     *
     * @param arg
     */
    void process(Object[] arg);
}
```
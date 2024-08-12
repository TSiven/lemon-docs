## 简介

`lemon-utils`是一个Java工具类库，通过静态方法封装，降低相关API的学习成本，它节省了开发人员对项目中公用类和公用工具方法的封装时间，使开发专注于业务，同时可以最大限度的避免封装不完善带来的bug。

> 框架包集成[Hutool-core](https://hutool.cn/docs/)工具集，lemon-utils亦是在此基础上进行延伸扩展

- [Hutool Doc](https://hutool.cn/docs/)

## 安装

1. 在项目的`pom.xml`中添加`parent`父依赖

```xml
<parent>
    <groupId>com.lemon.framework</groupId>
	<artifactId>lemon-dependencies</artifactId>
    <version>1.0.0.RELEASE</version>
</parent>
```

2. 在项目`pom.xml`添加`dependencies`引用

```xml
<dependency>
    <groupId>com.lemon.framework</groupId>
    <artifactId>lemon-utils</artifactId>
</dependency>
```

## JavaBean

### BeanConvertUtil

#### 概述

Object对象的转换复制功能，便于开发过程中的各个领域模型的转换传输过程。

#### 方法

##### 根据类型构造对象

```java
User user = BeanConvertUtil.newInstance(User.class)
```

##### List Bean 转换 List Map

> List<T>转换为List<Map<String, Object>>

```java
List<User> users = Lists.newArrayList();
...

List<Map<String, Object>> listMap = BeanConvertUtil.toListMap(users);
```

##### List Map 转换 List Bean

> List<Map<String,Object>>转换为List<R>

```java
List<Map> maps = Maps.newHashMap();
...

List<User> listMap = BeanConvertUtil.convertListMap(maps, User.class);
```

##### 转换集合对象并拷贝相应属性

> 将当前列表转换成另外指定对象类型的列表，并拷贝对象字段

- 方式1：对象方式拷贝

```java
List<User> users = Lists.newArrayList();
...
  
//字段1, 字段1属性为可变长度的参数, 作用于拷贝过程中忽略的字段内容
List<UserVo> listMap = BeanConvertUtil.convertListMap(maps, UserVo.class, "字段1" , "字段1");
```

- 方式2：指定属性拷贝(实现convert接口)

```java
List<User> users = Lists.newArrayList();
...
  
List<UserVo> listMap = BeanConvertUtil.convertListMap(maps, convert->{
		UserVo vo = new UserVo();
  	vo.setUsername(convert.getUsername());
  	vo.setNickname(convert.getNickname());
  	....
    
  	return vo;  
});
```

##### 克隆List对象

```java
List<User> users = Lists.newArrayList();
...
  
List<User> newUsers = BeanConvertUtil.cloneList(users);
```

##### 转换对象并拷贝相应属性

> 将当前对象转换成另外指定对象类型的列表，并拷贝对象字段

- 方式1：对象方式拷贝

```java
User user = new User();
...

//字段1, 字段1属性为可变长度的参数, 作用于拷贝过程中忽略的字段内容
UserVo userVo = BeanConvertUtil.convertObject(user , UserVo.class , "字段1" , "字段1");
```

- 方式2：注定属性拷贝(实现convert接口)

```java
User user = new User();
...
  
UserVo userVo = BeanConvertUtil.convertObject(user , convert->{
    UserVo vo = new UserVo();
  	vo.setUsername(convert.getUsername());
  	vo.setNickname(convert.getNickname());
  	....
      
  	return vo;  
});
```

## 日期时间

### DateTimeUtil

#### 概述

提供针对JDK8中LocalDate和LocalDateTime对象的封装。

#### 方法

```java
    /**
 * 时间戳(秒)转时区
 *
 * @param timestamp      时间戳
 * @param originalZoneId 原始时区的ZoneId
 * @param offset         偏移量
 * @return
 */
public static long convertTimestampSecondTo(long timestamp, ZoneId originalZoneId, ZoneOffset offset);


public static ZoneOffset toZoneOffset(ZoneId zoneId);
```

### ExpireTimeUtil

#### 概述

提供对过期时间的对象封装，根据过期时间的策略计算过期时间。

#### 过期类型枚举

```java
/**
 * 过期类型 <br>
 * <br>
 *
 * @author siven
 * @Date 2019-07-05 15:22
 */
public enum ExpireTypeEnum {

    /**
     * 年
     */
    YEAR,

    /**
     * 季度
     */
    QUARTER,

    /**
     * 月
     */
    MONTH,

    /**
     * 周
     */
    WEEK,

    /**
     * 日
     */
    DAY,

    /**
     * 永不过期
     */
    NEVER,

    /**
     * 默认
     */
    DEFAULT
}
```

#### 日期单位

```java
/**
 * 日期时间单位，每个单位都是以毫秒为基数
 * @author Looly
 *
 */
public enum DateUnit {
	/** 一毫秒 */
	MS(1), 
	/** 一秒的毫秒数 */
	SECOND(1000), 
	/**一分钟的毫秒数 */
	MINUTE(SECOND.getMillis() * 60),
	/**一小时的毫秒数 */
	HOUR(MINUTE.getMillis() * 60),
	/**一天的毫秒数 */
	DAY(HOUR.getMillis() * 24),
	/**一周的毫秒数 */
	WEEK(DAY.getMillis() * 7);
	
	private long millis;
	DateUnit(long millis){
		this.millis = millis;
	}
	
	/**
	 * @return 单位对应的毫秒数
	 */
	public long getMillis(){
		return this.millis;
	}
}
```

#### 方法

##### 获取当前日期剩余过期时间

```java
//当前日期距离当天结束剩余秒
long expireTime = ExpireTimeUtil.get(ExpireTypeEnum.DAY , DateUnit.SECOND);

//当前日期距离当年结束剩余天
long expireTime = ExpireTimeUtil.get(ExpireTypeEnum.YEAR , DateUnit.DAY);
```
##### 获取剩余过期时间

```java
//当前日期距离当年结束剩余天
long expireTime = ExpireTimeUtil.get(new Date(), ExpireTypeEnum.YEAR , DateUnit.DAY);
```

## Spring

### SpelUtil

#### 概述

Spring 表达式语言 Spring Expression Language（简称 SpEL ）是一个支持运行时查询和操作对象图的表达式语言 。 语法类似于 EL 表达式 ，但提供了显式方法调用和基本字符串模板函数等额外特性。该工具类Spring的基础上, 封装了SpEL解析，提升易用性。

### SpringContextUtil

#### 概述
Spring Context 对象，可以通过该工具类获取Spring Bean，以及发布事件

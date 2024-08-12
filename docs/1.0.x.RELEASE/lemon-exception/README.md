## 简介

​    `lemon-exception`是框架提供异常归集包，提供给业务使用，方便系统对异常进行区分，统一拦截；

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
    <artifactId>lemon-exception</artifactId>
</dependency>
```

## 如何使用

### 异常类说明

| 异常类                                                    | 描述               |
|--------------------------------------------------------|------------------|
| com.lemon.framework.exception.BaseException       | 错误基类             |
| com.lemon.framework.exception.ValidationException | 验证异常，在业务验证不通过时抛出 |
| com.lemon.framework.exception.AssertException     | 断言异常             |

### 业务断言
`Class: com.lemon.framework.exception.AssertUtil`

| NO | 方法                                                                          | 描述          |
|----|-----------------------------------------------------------------------------|-------------|
| 1  | void assertTrue(boolean express, String msg, String... args)                | 断言真, 非真抛出异常 |
| 2  | void assertNotEmpty(Object object, String... args)                          | 断言对象不对空     |
| 3  | void assertEmpty(Object object, String msg, String... args)                 | 断言对象为空      |
| 4  | void assertNotEmpty(Object object, ApiEnum<String> retCode, String... args) | 重载方法        |
| 5  | void assertEmpty(Object object, ApiEnum<String> retCode, String... args)    | 重载方法        |
| 6  | void assertTrue(boolean express, ApiEnum<String> retCode, String... args)   | 重载方法        |

> 以上断言方法根据场景进行了一些重载，主要针对Result.code进行额外的错误码区分。
>
> 在默认情况下，Result.code则分别为以下几种:

- assertTrue：405 (非法的操作请求)
- assertNotEmpty: 409 (数据已存在)
- assertEmpty：406 (数据不存在)
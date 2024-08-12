## 简介

​    `lemon-api`是后端开WEB发的一种规范。在前后端分离的体系下，所有接口响应结果保持高度一致性，也是使用框架包必需要遵循的一种约定。主要对以下几点内容进行说明：

- 接口报文格式风格一致性
- 返回状态码规范
- 枚举接口及常用枚举定义

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
    <artifactId>lemon-api</artifactId>
</dependency>
```

## 响应格式

后端返回的报文统一采用JSON方式，定义如下：

- 对象响应：

```json
{
    # 状态码
    code: 200,
    # 信息描述
    msg: "success",
    # 跟踪ID
    traceId: '42c66deef3900f98',
    # 时间戳
    timestamp: 201923203923023232,
    # 返回值
    data: {}
}
```

- 集合响应(含分页)

```json
{
    # 状态码
    code: 200,
    # 信息描述
    msg: "success",
    # 跟踪ID
    traceId: '42c66deef3900f98',
    # 时间戳
    timestamp: 201923203923023232,
    # 返回值
    data: [],
    # 总记录数
    total: 50,
    # 页码
    pageNo: 1,
    # 分页数量
    pageSize: 10,
}
```

### 报文描述

#### code

对请求的状态进行标识

#### msg

请求接口发生错误时，对错误的内容友好的进行提示。主要跟retCode配合使用

#### traceId

请求接口的跟踪ID

#### timestamp

请求接口响应的当前时间戳

#### data

返回数据体，JSON格式，根据不同的业务又不同的JSON体。

#### total

请求分页数据的总记录数

#### pageNo

请求分页数据的页码

#### pageSize

请求分页数据的页数

### 结果定义

#### com.lemon.framework.api.data.ResultBase

ResultBase<T>是一个基础类，主要定义了基础要素核常用的方法定义，比如isSuccess(), isFailed()以及对结果的断言方法

#### com.lemon.framework.api.data.Result

对ResultBase类的扩展，加入了一些静态方法，方便使用构造Result对象；

#### com.lemon.framework.api.data.ResultPage

与Result作用类似，另外Result基础上对data属性泛型了集合对象，同时会返回跟分页相关的属性字段；

### 其他

#### com.lemon.framework.api.data.BaseDataObject

POJO对象超级基类，所有POJO对象都需要继承该类，主要实现了序列化，方便后续框架工具的封装。

#### com.lemon.framework.api.data.PageVo

分页请求对象定义，使用分页场景时，可以继承该对象

## 状态码规范

> 状态码“0”表示请求成功
> PS: 通用框架定义错误码在`com.lemon.framework.api.ResultCode`类中维护,

### 系统通用错误码

| <nobr>错误码</nobr> | 错误描述                | 场景说明                              |
|------------------|---------------------|-----------------------------------|
| 200              | success             | 成功                                |
| 500              | Unknown Error       | 多数是由未知异常引起的，仔细检查传入的参数是否符合文档描述     |
| 502              | Bad Gateway         | 调用其他第三方(feign)服务发生错误              |
| 503              | Too Many Requests   | 重复请求某资源导致操作频繁错误                   |
| 400              | Bad Request         | 请求出现语法错误                          |
| 405              | Illegal Operation   | 请求资源出现非正常或不满足条件时的操作异常             |
| 406              | Data Not Exists     | 获取资源所需要的策略并没有被满足                  |
| 408              | Request Timeout     | 网关服务器尝试执行请求时，未能及时从上游服务器           |
| 409              | Data already exists | 由于数据已经存在，服务器无法存储完成请求的内容           |
| 412              | Invalid Parameter   | 一般是用户传入参数非法引起的，请仔细检查入参格式、范围是否一一对应 |
| 429              | Request limit       | 由于服务器负载过高，对请求进行限流处理               |

### 业务错误码规范

状态码主要由4位数字组成。各个业务系统返回必须包含系统缩写作为前缀。比如，(代理系统)`AGT_XXXX`、（OTC）`OTC_XXXX`、（网关）`GW_XXXX`
。方便后续快速跟踪系统异常是什么系统抛出

## 枚举规范

所有的枚举类必须实现ApiEnum接口，枚举定义规范如下：

```java
public interface ApiEnum<T> {

    static <T extends Enum<T> & ApiEnum<T>> T valueOf(String value, Class<T> clazz) {
        T enumm = Enum.valueOf(clazz, value);
        return enumm;
    }

    /**
     * 根据值获取对应的枚举
     *
     * @param enumClazz 枚举的类型
     * @param value     枚举的值
     * @param <Z>       枚举值的类型的泛型
     * @param <R>       枚举类型的泛型
     * @return
     * @throws Exception
     */
    static <Z, R extends ApiEnum<Z>> R parse(Class<R> enumClazz, Z value) {
        if (value != null) {
            R[] values = null;
            try {
                Method method = enumClazz.getMethod("values");
                values = (R[]) method.invoke(null);
            } catch (Exception e) {
                throw new IllegalArgumentException(e);
            }

            for (R each : values) {
                if (value.equals(each.getValue())) {
                    return each;
                }
            }
        }
        return null;
    }


    /**
     * 根据值获取对应的枚举的描述
     *
     * @param enumClazz 枚举的类型
     * @param value     枚举的值
     * @param <Z>       枚举值的类型的泛型
     * @param <R>       枚举类型的泛型
     * @return 值对应的枚举的描述
     */
    static <Z, R extends ApiEnum> String parseToText(Class<R> enumClazz, Z value) {
        ApiEnum apiEnum = parse(enumClazz, value);
        if (apiEnum != null) {
            return apiEnum.getText();
        }
        return null;
    }

    /**
     * 根据描述获取对应的枚举
     *
     * @param enumClazz 枚举的类型
     * @param text      枚举的描述
     * @param <R>       枚举类型的泛型
     * @return
     * @throws Exception
     */
    static <R extends ApiEnum> R parseText(Class<R> enumClazz, String text) {
        if (text != null) {
            R[] values = null;
            try {
                Method method = enumClazz.getMethod("values");
                values = (R[]) method.invoke(null);
            } catch (Exception e) {
                throw new IllegalArgumentException(e);
            }

            for (R each : values) {
                if (text.equals(each.getText())) {
                    return each;
                }
            }
        }
        return null;
    }

    /**
     * 根据枚举值判断枚举是否存在
     *
     * @param enumClazz 枚举的类型
     * @param value     枚举的值
     * @param <Z>       枚举值的类型的泛型
     * @param <R>       枚举类型的泛型
     * @return 枚举值存在返回true，否则返回false
     */
    static <Z, R extends ApiEnum> boolean isValueValid(Class<R> enumClazz, Z value) {
        return parse(enumClazz, value) != null;
    }

    /**
     * 获取值
     */
    T getValue();

    /**
     * 获取描述
     */
    String getText();
}
```
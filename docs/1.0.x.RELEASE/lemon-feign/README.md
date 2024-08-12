## 简介

`lemon-feign`是对spring-cloud-starter-openfeign的扩展，主要扩展以下能力：

- feign info日志格式输出规范
- 扩展feign get请求无法使用实体类传输问题
- feign调用链路跟踪

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
    <artifactId>lemon-feign-spring-boot-starter</artifactId>
</dependency>
```

## 日志规范

基于Feign日志工厂进行改造，继承`feign.Logger`接口，重写了logRequest()、logAndRebufferResponse()方法，最终产生的日志格式如下：

```verilog
[FeignClient#getXXXXXX] ---> GET http://cnt-customer-service/query/relation_customer_info/CNO2019113010210001/BIZ_SUBJECT_COMPANY HTTP/1.1
[FeignClient#getXXXXXX] HEADERS: {traceId=[5debfdbee4b0947b2d297ec8], app=[work-order-service], Accept-Encoding=[gzip, deflate]}
[FeignClient#getXXXXXX] ---> END HTTP (0-byte body)
[FeignClient#getXXXXXX] <--- HTTP/1.1 200  (7ms)
[FeignClient#getXXXXXX] HEADERS: {content-type=[application/json;charset=UTF-8], date=[Mon, 09 Dec 2019 07:35:00 GMT], transfer-encoding=[chunked]}
[FeignClient#getXXXXXX] {"traceId":"5debfdbee4b0947b2d297ec8","retCode":"CU1001","retMsg":"无此关系客户","timestamp":1575876900149}
[FeignClient#getXXXXXX] <--- END HTTP (113-byte body)
```

## GET请求实体类传输

在spring cloud feign client中，使用`@SpringQueryMap`传递参数

```java
import org.springframework.cloud.openfeign.SpringQueryMap;
/**
 * 获取经纪人管理列表(运营平台)
 *
 * @param reqVo
 * @return
 */
@GetMapping("/broker_list")
ResultPage<AgentUserVo> findByCondition(@SpringQueryMap BrokerQryReqVo reqVo);
```
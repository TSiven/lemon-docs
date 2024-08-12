# CHANGELOG


## [1.0.0.RELEASE] 2024/06/21
### 更新组件
- job
    `[新特性]`xxl-job执行日志扩展日志追踪traceId打印

- lemon-i18n
  `[优化]`ApiEnum枚举输出文本i18n优化，默认根据类名.枚举name动态获取i18n定义资源KEY

- lemon-web
  `[优化]`@RequestHeader， @TimestampDeserializer 由于HandlerMethodArgumentResolver执行顺序，导致自定义解析不生效, 提高执行顺序优化


---

## [1.1.2.RELEASE] 2024/06/12
### 更新组件
- lemon-validator
    - `[新特性]`新增`@CompareFields`
      校验注解，支持多字段对比关联校验. [详见](http://dbex.gitlab01.bvsprd.com/krypton/base-component/lemon-docs/#/1.0.x.RELEASE/lemon-validator/README?id=%e6%96%b9%e5%bc%8f%e4%b8%89%ef%bc%9a%e4%bd%bf%e7%94%a8%e8%87%aa%e5%ae%9a%e4%b9%89%e6%b3%a8%e8%a7%a3comparefield%e6%af%94%e8%be%83%e5%ad%97%e6%ae%b5%e5%a4%a7%e5%b0%8f)
- lemon-exception
    - `[优化]`GlobalExceptionHandler参数校验异常提示语优化
- lemon-web
    - `[优化]`日志组件`@LogAnnotation`描述description参数支持SpEL动态参数解析，拦截器支持组合注解合并解析。
    - `[优化]`请求Header注入组件`@RequestHeader`新增默认值、必传校验特性

---

## [1.1.1.RELEASE] 2024/01/27
### 更新组件
- lemon-web
    - `[bugfix]`jackson序列化偶发失效BUG修复

---

## [1.1.0.RELEASE] 2024/01/27
### 更新组件

- lemon-web
    - `[bugfix]`表单重复提交组件`@RequestLimiter`异常捕获后没有抛出导致消息无法正常响应、执行方法切面不生效
    - `[新特性]`新增`@DomainSerializer`序列化，支持动态替换域名资源
    - `[新特性]`新增`@NumberToStringSerializer`Number 转 String序列化
    - `[新特性]`新增`@LongToStringSerializer`Long to String序列化
    - `[新特性]`新增`@DesensitizationSerializer`响应数据脱敏
- lemon-test
    - `[优化]`支持静态方法mock
- lemon-utils
    - `[优化]`DateTimeUtil新增系统时区转UTC8时区方法

---

## [1.0.9.RELEASE] 2023/12/27
### 更新组件
- lemon-i18n
    - `[新特性]`响应前端枚举类型，根据用户语言，动态转成对应语言的文本
- lemon-test
    - `[新特性]`单元测试支持
- 其他：
  - MissingRequestCookieException / MissingRequestHeaderException 异常捕获支持
  - DateTimeUtil 时间戳系统时区转UTC8时区工具类支持


## [1.0.8.RELEASE] 2023/12/21
### 更新组件
- lemon-logger
  - `[新特性]`支持请求的参数，以及响应结果报文进行格式化日志输出
  - `[新特性]`支持调用方法日志拦截存储
  - `[新特性]`支持日志全链路跟踪
- lemon-web
  - `[新特性]`提供自定义序列化、反序列化接口（com.lemon.framework.web.jackson.ValueJsonCustomizer），业务系统根据实际需求扩展
  - `[新特性]`表单重复提交、接口限流的支持，避免后端因为重复提交内容导致异常问题
- lemon-api
  - `[优化]`Result、ResultBase、ResultPage相关断言成功`assertSuccess()`方法采用链式形参返回实体对象本身

## [1.0.7-RELEASE] 2023/11/25
### BUGFIX
- lemon-web
  - 修复序列化时间前端传递用户时区Header(`X-User-Timezone`) 值被URLEncoder，导致序列化处理异常


## [1.0.6.RELEASE] 2023/11/18
### 更新组件
- lemon-api \ lemon-web
    - 支持序列化包含空值字段配置，默认不包含
```yml
  lemon:
    # WEB配置
    web:
      # 序列化配置
      serialization:
        include: always
```


## [1.0.5.RELEASE] 2023/11/06
### 更新组件
- lemon-api \ lemon-web
    - com.lemon.framework.api.jackson.serializer.SerializerConvert 序列化日期转换类移动到lemon-api中维护

## [1.0.4.RELEASE] 2023/10/25
### 更新组件
- lemon-api \ lemon-feign \ lemon-all-spring-boot-starter
    - sentinel、loadbalancer pom 移动到lemon-feign中维护


## [1.0.3.RELEASE] 2023/10/24
### 更新组件
- lemon-api \ lemon-exception \ lemon-i18n \ lemon-validator
    - bugfix: 解决国际化message返回为code的问题 siven 2023/10/18, 15:20 
    - SCA依赖组件版本漏洞升级，影响：hutool-core\snakeyaml\commons-beanutils siven 2023/10/20, 11:46 
    - 引入flatten-maven-plugin插件，解决pom version问题 
    - lemon-api移除 hutool-core siven 2023/10/21, 01:09 
    - 私仓地址修改为新的域名：http://nexus.bvsprd.com siven Yesterday 09:54 
    - RequestHolder 提供 getLocale 方法 siven Yesterday 11:56


## [1.0.2.RELEASE] 2023/10/11
### 更新组件
- lemon-api \ lemon-exception \ lemon-i18n \ lemon-validator
  - 移除PageSize分页最大限制200条
  - Result Code 类型调整为int类型 siven 29 minutes ago


## [1.0.1.RELEASE] 2023/10/09
### 更新组件
- lemon-api
  - PageSize分页最大限制200条
  - 新增ResultUtil类，处理Result对象相互转换 siven 2023/9/25, 00:31
  - Result实例化优化，无参构造函数集成父类实例空对象 siven 2023/9/27, 16:31
  - Result 优化 isSuccess() 判断， 兼容0的情况 siven 2023/9/27, 19:55
- lemon-exception
  - 新增全局异常拦截处理开关 siven 2023/9/21, 20:23
  - HTTP媒体类型不支持的异常捕获 siven 2023/9/22, 22:05
- lemon-starter
  - 集成 alibaba sentinel 组件，支持熔断策略 siven 2023/9/22, 02:31
- lemon-web
  - String类型参数反序列化处理trim无效空格 siven 2023/9/21, 23:07
  - Long 长度超过16位 to String 序列化 siven 2023/9/25, 23:07
  - TimestampSerializer 支持自定义format siven Yesterday 17:44



## [1.0.0.RELEASE] 2023/9/15
### 更新组件
  - lemon-api
  - lemon-cache
  - lemon-database
  - lemon-dependencies
  - lemon-exception
  - lemon-feign
  - lemon-i18n
  - lemon-logger
  - lemon-starter
  - lemon-utils
  - lemon-validator
  - lemon-web

---

## [v模板] yyyy.MM.dd

### 更新组件

- bc-logger
- bc-utils

### 更新内容

- 日志`@LogAnnotation`注解扩展参数`key`, 支持`SEpl`获取请求参数, 返回参数值
- 日志类型改为String, 便于系统扩展

---
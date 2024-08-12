## 简介

`lemon-web`是Spring Web的扩展工具集，主要提供了以下支持：

- Jackson序列化扩展
- WEB工具包
- 同一响应内容体
- RequestBody扩展
- 接口防重复提交 `1.0.9.RELEASE` 以上版本支持

## 安装

1. 在项目的`pom.xml`中添加`parent`父依赖

```xml
<parent>
    <groupId>com.lemon.framework</groupId>
	<artifactId>lemon-dependencies</artifactId>
    <version>1.1.3.RELEASE</version>
</parent>
```

2. 在项目`pom.xml`添加`dependencies`引用

```xml
<dependency>
    <groupId>com.lemon.framework</groupId>
    <artifactId>lemon-web-spring-boot-starter</artifactId>
</dependency>
```

## 扩展配置

```yaml
# Config
  lemon:
    # WEB配置
    web:
      # 序列化配置
      serialization:
        # 是否包含null值属性。不包含(NON_NULL)，包含（always）
        include: non_null
        domain: https://static.bivbris.com # 动态替换域名资源 `1.1.3.RELEASE` 以上版本支持
        enable-sensitive: true # 是否开启脱敏 `1.1.3.RELEASE` 以上版本支持
      # 日期格式化配置
      date-format:
        serializer-to-user-time-zone: true # 序列化：系统时区TO 用户时区(默认不开启)
        deserializer-to-system-time-zone: true # 反序列化：用户时区TO 系统时区(默认不开启)
        user-time-zone-header: X-User-Timezone # 用户时区Header参数名
        default-user-time-zone: Asia/Shanghai # 默认用户时区
        system-time-zone: UTC # 系统时区
      # 接口就重复请求配置 `1.0.9.RELEASE` 以上版本支持
      limit:
        enable: true # 开关
        timeout: 3 # 多少秒内每个用户只允许访问一次
        time-unit: seconds # 时间单位
        cache-mode: redis # 缓存模式, 默认redis
        repeat-key-generator: com.lemon.framework.web.limit.keygen.DefaultRepeatLimitKeyGenerator # 全局防重复提交默认Key生成器
        rate-key-generator: com.lemon.framework.web.limit.keygen.DefaultRateLimitKeyGenerator # 全局速率限流默认Key生成器
        permits: 50 # 速率限流可处理的请求数量
```

## 如何使用

### Jackson序列化

Jackson是一个 Java 用来处理 JSON 格式数据的类库，其性能非常好。Jackson具有比较高的序列化和反序列化效率，spring web默认则采用该序列化工具对资源请求参数以及返回参数进行序列化处理，框架在此基础上延伸扩展；

- 支持日期类型，包括JAVA8(LocalDate、LocalDateTime)、Date以及时间戳(timestamp)的序列化，反序列化
- ApiEnum 枚举序列化、反序列化

#### 日期序列化
日期处理支持针对用户所在的地区分区。动态序列化、反序列化显示目标的时区。通过Header参数控制。
- Header： `X-User-Timezone`: 地区（America/Anchorage）

![img.png](_media/img.png ':size=40%')

##### timestamp (时间戳)

###### 序列化
- 注解：`com.lemon.framework.api.jackson.serializer.TimestampSerializer`
- 处理类：`com.lemon.framework.web.jackson.serializer.TimestampToUserZoneJsonSerializer`
- 说明：时间戳TO System Time Zone
- 示例代码：

```java
import com.lemon.framework.api.jackson.serializer.TimestampSerializer;

public class ExampleVo extends BaseDataObject {
    private static final long serialVersionUID = 1L;
    
    /**
     * 创建时间 (代理开通时间)
     * 1.0.1.RELEASE 版本支持format格式化定义
     */
    @TimestampSerializer(format = "yyyy-MM-dd")
    private Long createdAt;
    /**
     * 更新时间
     */
    @TimestampSerializer
    private Long updatedAt;
}

```

- 响应结果：
根据返回的参数名 + `Convert`，返回转换结果对象，如下：
```json
{
  ...,
  "createdAt": 1694196065888,
  "createdAtConvert": {
    "format": "2023-09-08" # 根据用户地区动态转换时区时间
  }
  "updatedAt": 1694196065888,
  "updatedAtConvert": {
    "format": "2023-09-08 10:01:05" # 根据用户地区动态转换时区时间
  }
}
```


###### 反序列化
- 注解：`com.lemon.framework.api.jackson.deserializer.TimestampDeserializer`
- 处理类：`com.lemon.framework.web.jackson.deserializer.TimestampToSystemZoneDeserializer`
- 示例代码：

```java
import com.lemon.framework.api.jackson.deserializer.TimestampDeserializer;

@Getter
@Setter
public class ExampleVo extends BaseDataObject {

    /**
     * 创建时间（开始时间）
     */
    @TimestampDeserializer
    Long createdStartTime;

    /**
     * 创建时间（结束时间）
     */
    @TimestampDeserializer
    Long createdEndTime;
}
```

##### 其他日期类型

###### 序列化 
日期类型(Date、LocalDate、LocalDateTime）序列化时，会新增一个字段，将时间戳、目标时区格式化时间进行返回，返回字段格式：
```json
{
  ...
  "updated": "2023-12-12 12:12:00",
  "updatedConvert": {
    "timestamp": 1694196065888,
    "format": "2023-12-12 20:12:00" # 根据用户地区动态转换时区时间
  }
}
```

###### 反序列化
- 日期类型支持接收前端传递`yyyy-MM-dd`或`yyyy-MM-dd HH:mm:ss`等明文文本格式反序列化解析成对应的类型（Date、LocalDate、LocalDateTime）
- 开启日期反序列化为系统时区开关 `lemon.web.date-format.deserializer-to-system-time-zone`，系统会根据用户的时区自动转换为系统UTC时区的日期。 

---

#### 域名响应序列化

`1.1.3.RELEASE` 以上版本支持

由于S3域名资源有被墙的风险, 存储数据库内容会将域名存储到库中, 并非存储资源ID, 返回到前端域名失效。 该工具提供根据域名资源内容, 动态替换域名返回至前端。

> 域名配置方式说明：
> - 通过`yaml`配置定义默认域名配置`lemon.web.serialization.domain`
> - 使用 `@DomainSerializer`的`value`指定域名
> - 实现接口：`com.lemon.framework.web.jackson.DomainProvider`，提供域名信息。通过 `@DomainSerializer`的`provider`指定Spring Bean


##### 注解使用示例
```java
@Data
@Slf4j
public class ShareTemplateVo{
    /**
     * 默认海报URL，读取lemon.web.serialization.domain配置
     */
    @DomainSerializer
    private String posterUrl;
    
    /**
     * 默认海报URL, 指定替换的域名
     */
    @DomainSerializer("https://www.baidu.com")
    private String posterUrl2;

    /**
     * 默认海报URL，指定DomainProvider的Spring Bean
     * 支持List String
     */
    @DomainSerializer(provider = "customDomainProvider")
    private List<String> posterUrls;
}
```

##### 实现获取域名接口
```java
@Component("customDomainProvider")
public class CustomDomainProvider implements DomainProvider {
    @Override
    public String get() {
        return "https://www.bitventus.com";
    }
}
```

#### 数据脱敏序列化
> `1.1.3.RELEASE` 以上版本支持

实现内容脱敏展示，期望做到可灵活配置，灵活启用，并且最好内置丰富插件，支持手机号、邮箱、身份证号、住址、中文名、座机号、银行卡、自定义等多种类型的脱敏配置。

##### 开启脱敏注解
```yaml
  lemon:
    # WEB配置
    web:
      # 序列化配置
      serialization:
        enable-sensitive: true # 是否开启脱敏 `1.1.3.RELEASE` 以上版本支持
```


##### 脱敏注解使用
```java
public class SensitiveEntity{
    @DesensitizationSerializer(SensitiveTypeEnum.PASSWORD)
    String password = "12334343";

    @DesensitizationSerializer(SensitiveTypeEnum.EMAIL)
    String email = "12334343@qq.com";
}
```

##### 脱敏策略说明
- `NAME`: 中文名, 只显示第一个汉字，其他隐藏为2个星号，比如：李**
- `ID_CARD`： 身份证号, 前1位 和后2位
- `FIXED_PHONE`: 固定电话, 前四位，后两位
- `MOBILE_PHONE`: 手机号, 前三位，后4位，其他隐藏，比如135****2210
- `ADDRESS`: 地址,只显示到地区，不显示详细地址，比如：北京市海淀区****
- `EMAIL`: 电子邮件, 仅显示第一个字母，前缀其他隐藏，用星号代替，@及后面的地址显示，比如：d**@126.com,
- `BANK_CARD`: 银行卡, 由于银行卡号长度不定，所以只展示前4位，后面的位数根据卡号决定展示1-4位
- `PASSWORD`: 密码, 全部字符都用*代替，比如：******
- `FIRST_MASK`: 只显示第一个字符,
- `CLEAR_TO_NULL`: 清空为null,
- `CLEAR_TO_EMPTY`: 清空为"",
- `CUSTOM`: 自定义脱敏策略, 配合注解的`startInclude`、`endExclude`属性使用

##### 返回结果
```json
{
  "code": 200,
  "msg": "success",
  "traceId": "218d5c19444eccd2",
  "timestamp": 1706455024657,
  "data": {
    "password": "********",
    "email": "1*******@qq.com"
  }
}
```


#### 数值序列化
> `1.1.3.RELEASE` 以上版本支持

实现对响应的数值类型转String，支持对小数点保留处理、以及格式化内容输出

##### 注解使用
```java
public class NumberEntity{
    @NumberToStringSerializer(format = "#,##0.00")
    BigDecimal bigDecimal = new BigDecimal("10002000.1235");

    @NumberToStringSerializer(scale = 2)
    BigDecimal bigDecimal2 = new BigDecimal("10002000.1235000");

    @NumberToStringSerializer
    BigDecimal bigDecimal3 = new BigDecimal("10002000.1235000");

    @NumberToStringSerializer(scale = 3, roundingMode = RoundingMode.HALF_UP)
    BigDecimal bigDecimal4 = new BigDecimal("10002000.1235000");

    @NumberToStringSerializer(stripTrailingZeros = false)
    BigDecimal bigDecimal5 = new BigDecimal("10002000.1235000");

    @NumberToStringSerializer(format = "#,##0.00")
    double double1 = 10002000.1235d;
}
```


#### 枚举序列化
- 枚举类型支持针对用户选择语言，动态将枚举描述内容返回显示
- 枚举必须实现`com.lemon.framework.api.ApiEnum`接口

##### 序列化
- Header： `Accept-Language`: 语言 （zh_CN）

- 序列化类：com.lemon.framework.web.jackson.serializer.ApiEnumJsonSerializer
- 枚举类型(ApiEnum）序列化时，会新增一个字段，将枚举定义的Text进行返回，返回字段示例：

```json
{
  "status": 1,
  "statusConvert": {
    "format": "有效"
  }
}
```

##### 反序列化
传入参数支持传递枚举的Value、names()的解析

- 示例如下：
```java
public enum AgentStatusEnum implements ApiEnum<Integer> {

    EFFICIENT(1, "有效"),

    INVALID(2, "无效");

    int value;

    String text;

    AgentStatusEnum(int value, String text) {
        this.value = value;
        this.text = text;
    }

    public static AgentStatusEnum parse(int value) {
        try {
            return ApiEnum.parse(AgentStatusEnum.class, value);
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    public Integer getValue() {
        return this.value;
    }

    @Override
    public String getText() {
        return this.text;
    }
}
```

- 使用`StatusEnum`定义的Value Code情况解析示例：

![img.png](_media/img_11.png ':size=60%')


- 使用`StatusEnum`定义的Enum 定义names()情况解析示例：

![img.png](_media/img22.png ':size=60%')


#### 其他
- `@LongToStringSerializer`: Long to String


#### 自定义全局序列化
提供自定义摸序列化接口，可根据项目实际序列化需求，自定义`Module`扩展 (1.0.8以上版本支持)

```java
package com.lemon.framework.web.jackson;

/**
 * 自定义序列化、反序列化接口
 *
 * @Author: siven
 */
public interface ValueJsonCustomizer {

    /**
     * 创建自定义模块的序列化包装
     *
     * @return
     */
    Module createModuleCustomizer();
}

```

#### 自定义注解
提供根据自定义注解进行序列化，反序列化扩展功能，实现注解拦截序列化接口: `com.lemon.framework.web.jackson.AnnotationIntrospectorProvider` (1.1.0以上版本支持)

##### 注解定义

```java
/**
 * 自定义序列化
 *
 * @Author: siven
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface CustomSerializer {

    String value() default "";
}
```

##### 实现注解序列化拦截接口

```java
/**
 * 自定义注解序列化拦截实现
 *
 * @author siven
 */
@Slf4j
public class CustomJsonSerializer extends JsonSerializer<String> implements AnnotationIntrospectorProvider<CustomSerializer> {

    @Override
    public Class<? extends Annotation> support() {
        return CustomSerializer.class;
    }

    //如果是进行序列化处理， 实现序列化接口，反序列化则实现 JsonDeserializer
    @Override
    public JsonSerializer serializer(CustomSerializer annotated) {
        //设置注解定义属性
        this.annotatedValue = annotated.value();
        return this;
    }

    //注解定义属性
    String annotatedValue;

    //jackson序列化调用方法
    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializerProvider) throws IOException {
        if (Objects.isNull(value)) {
            gen.writeNull();
            return;
        }

        //对字段值进行处理
        gen.writeString(value + ": CustomJsonSerializer: " + annotatedValue);
    }
}
```

##### 配置实现类
在`resources`目录下新增`META-INF/services/com.lemon.framework.web.jackson.AnnotationIntrospectorProvider`文件，内容定义实现接口的全限量名，一行一个实现类。如下所示：
```java
com.lemon.sample.quickstart.service.CustomJsonSerializer
```

> [完整代码示例](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples/-/blob/master/lemon-sample-quickstart/src/main/java/com/lemon/sample/quickstart/service/CustomJsonSerializer.java)

### 接口限流处理

> `1.0.9.RELEASE` 以上版本支持

对接口的限流功能的封装，主要应用场景：
- 单位时间内防止重复提交
- 对某个接口进行单位时间内请求数量的限制

> 其功能实现都使用了缓存，其中缓存可选择redis和guava， 只有redis可以实现分布式，其他方式只能在单机上使用

#### 注解定义
```java
/**
 * 请求限流组件
 *
 * @Author: siven
 */
@Documented
@Retention(RUNTIME)
@Target({METHOD, TYPE})
public @interface RequestLimiter {

    /**
     * 限流模式
     *
     * @return
     */
    LimiterMode mode() default LimiterMode.REPEAT_LIMIT;

    /**
     * key设置，支持spEl表达式
     *
     * @return key
     */
    String key() default "";

    /**
     * <pre>请求key生成器Spring BeanName</pre>
     * <br>
     * 实现接口: {@link KeyGenerator}
     * <br>
     * 系统默认：{@link DefaultRepeatLimitKeyGenerator}
     * <br>
     * 注解配置keyGenerator优先于{@link RequestLimiterProperties#getRepeatKeyGenerator()}
     *
     * @return
     */
    String keyGenerator() default "";

    /**
     * 限流回调函数
     * 实现接口：{@link Fallback}
     *
     * @return
     */
    String fallback() default "";

    /**
     * 几秒内不允许重复访问
     * <br>
     * 为0时，默认使用全局配置 {@link RequestLimiterProperties#getTimeout()}
     * <p>接口重复提交模式： {@link LimiterMode#REPEAT_LIMIT} 生效/p>
     *
     * @return timeout
     */
    long timeout() default 0;

    /**
     * 时间单位
     * <br>
     * timeout 为0时不生效
     *
     * <p>接口重复提交模式： {@link LimiterMode#REPEAT_LIMIT} 生效/p>
     *
     * @return 时间单位
     */
    TimeUnit timeUnit() default TimeUnit.SECONDS;

    /**
     * 每秒可处理的请求数量
     * <p>接口速率限流模式： {@link LimiterMode#RATE_LIMIT} 生效/p>
     *
     * @return 请求数量
     */
    int permits() default 0;

    /**
     * 防止重复提交策略， 默认{@link ReleaseStrategy#COMPLETED},程序执行完成释放标识
     */
    ReleaseStrategy strategy() default ReleaseStrategy.COMPLETED;
}
```

枚举说明:

- `mode`: 限流模式:
    - REPEAT_LIMIT： 接口重复提交
    - RATE_LIMIT: 接口速率限流

- `strategy`: 重复提交标识释放支持两种策略：
    - COMPLETED： 程序执行完成释放重复提交标识
    - PERSISTENT: 在指定时间内请求将持续获得标识，将都不允许重复请求，直到过期后释放

---

#### 如何使用

在资源请求方法或类上, 添加`@RequestLimiter`注解

```java
@Slf4j
@RequestLimiter
@RestController
@RequestMapping("/repeat_limit")
public class RepeatLimitResource {

    /**
     * 场景1：类级别控制防重复提交
     *
     * @return
     */
    @SneakyThrows
    @GetMapping
    public Result test1() {
        Thread.sleep(5000L);
        return Result.success();
    }

    /**
     * 场景2：方法自定义防重复提交
     *
     * @return
     */
    @SneakyThrows
    @RequestLimiter(timeout = 6)
    @GetMapping("/test2")
    public Result test2() {
        Thread.sleep(3000L);
        return Result.success();
    }

    /**
     * 场景3：执行3秒之内请求过，不允许重复请求
     *
     * @return
     */
    @SneakyThrows
    @RequestLimiter(strategy = ReleaseStrategy.PERSISTENT, timeout = 3, timeUnit = TimeUnit.SECONDS, fallback = "customFallback")
    @GetMapping("/test3")
    public Result test3(@RequestParam(defaultValue = "123") String id) {
        return Result.success();
    }

    /**
     * 场景4：自定义keyGenerator生成规则
     *
     * @return
     */
    @SneakyThrows
    @RequestLimiter(keyGenerator = "customKeyGenerator")
    @GetMapping("/test4")
    public Result test4() {
        Thread.sleep(3000L);
        return Result.success();
    }

    /**
     * 场景5：配置Key
     *
     * @return
     */
    @SneakyThrows
    @RequestLimiter(key = "'test-' + #id")
    @GetMapping("/test5")
    public Result test5(@RequestParam String id) {
        Thread.sleep(3000L);
        return Result.success();
    }

    /**
     * 场景6：令牌桶模式，每5秒钟允许6个请求
     *
     * @return
     */
    @SneakyThrows
    @RequestLimiter(mode = LimiterMode.RATE_LIMIT, permits = 6, timeout = 5, keyGenerator = "customKeyGenerator")
    @GetMapping("/test6")
    public Result test6() {
        return Result.success();
    }

    @Component("customKeyGenerator")
    public class CustomKeyGenerator implements KeyGenerator {
        @Override
        public String generate(HttpServletRequest request, Object target, Method method, Object... params) {
            StringJoiner joiner = new StringJoiner(",");
            for (Object param : params) {
                joiner.add(param.getClass().getName());
            }
            return target.getClass().getName().concat(":").concat(method.getName()).concat(":")
                    .concat(joiner.toString());
        }
    }

    @Component("customFallback")
    public class CustomFallback implements Fallback {
        @Override
        public void exec(String key, ProceedingJoinPoint pjp, HttpServletRequest request) {
            log.info("customFallback key {}", key);
        }
    }
}
```
示例代码：[GIT](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples/-/blob/master/lemon-sample-quickstart/src/main/java/com/lemon/sample/quickstart/resource/RepeatLimitResource.java?ref_type=heads)

#### Key的生成规则

##### 1. 实现框架提供Key生成规则接口

`com.lemon.framework.web.limit.keygen.KeyGenerator`

- 接口定义如下：
```java
@FunctionalInterface
public interface KeyGenerator {
    /**
     * Generate a key for the given method and its parameters.
     *
     * @param request request
     * @param target  the target instance
     * @param method  the method being called
     * @param params  the method parameters (with any var-args expanded)
     * @return a generated key
     */
    String generate(HttpServletRequest request, Object target, Method method, Object... params);
}
```

- 实现代码示例：
```java
@Component("customKeyGenerator")
public class CustomKeyGenerator implements KeyGenerator {
    @Override
    public String generate(HttpServletRequest request, Object target, Method method, Object... params) {
        String sessionId = request.getSession().getId();
        StringJoiner joiner = new StringJoiner(",");
        for (Object param : params) {
            joiner.add(param.getClass().getName());
        }
        return target.getClass().getName().concat(":").concat(method.getName()).concat(":").concat(joiner.toString());
    }
}
```

- 配置方式：
1. 注解方式指定： `@RepeatLimit(keyGenerator = "customKeyGenerator")`
2. 全局指定application config配置：`lemon.web.limit.key-generator=com.lemon.framework.web.limit.keygen.DefaultKeyGenerator`, 详见[配置](README?id=扩展配置)

> ⚠️注解`@RepeatLimit`配置`keyGenerator`优先于application config配置

##### 2. 为每个接口单独设置key，可以使用SpEL表达式
```java
/**
 * 场景5：配置Key
 *
 * @return
 */
@SneakyThrows
@RequestLimiter(key = "'test-' + #id")
@GetMapping("/test5")
public Result test5(@RequestParam String id) {
    Thread.sleep(3000L);
    return Result.success();
}
```

- 进阶：spEl扩展，通过EvaluationFiller扩展自定义全局SpEl变量

```java
public class SpElConfiguration {
    @Bean
    public EvaluationFiller evaluationFiller() {
        //context spEl上下文，target当前类实例，method当前调用方法，parameters方法的参数
        return ((context, target, method, parameters) -> {
            context.setVariable("className", target.getClass().getName());
            //配置好参数以后可以在spEl中使用，例如：@RepeatLimiting(key="#className + '-key'")
        });
    }
}
```

#### 限流函数回调

实现框架提供限流回调接口： `com.lemon.framework.web.limit.Fallback`

```java
@FunctionalInterface
public interface Fallback {
    void exec(String key, ProceedingJoinPoint pjp, HttpServletRequest request);
}
```


#### 响应结果

- 表单重复提交的情况下，系统拦截返回重复提交的消息如下

```json5
{
  "code": 503,
  "msg": "您的操作过于频繁，请稍后再试",
  "args": [],
  "traceId": "2d2f7c9bb0945bb8",
  "timestamp": 1702974876768
}
```

- 接口限流的情况下，系统拦截返回重复提交的消息如下：
```json5
{
  "code": 429,
  "msg": "系统繁忙，请稍后再试",
  "args": [],
  "traceId": "72ac974375d2954c",
  "timestamp": 1702978462566
}
```


### WEB工具包

#### RequestHolder

> com.lemon.framework.web.RequestHolder

Request持有类，作用于在任何地方获取Reuqest对象

### 统一响应内容体

````
作用于Resource响应内容, 统一包装`Result.class`对象
````

#### 使用说明

1. 在类或者方法上添加注解 `{@link com.lemon.framework.web.annotation.ResponseResultBody}`

#### 代码示例

```java
@RestController
@ResponseResultBody
@RequestMapping("response_result_body")
public class ResponseResultBodyResource {

    @Autowired
    UserService userService;

    @LogAnnotation(enableRepository = true)
    @ApiOperation("根据用户编号获取用户信息")
    @GetMapping("/{userNo}")
    public User getByUserNo(@PathVariable String userNo) {
        User user = userService.getByUserNo(userNo);
        return user;
    }

    @LogAnnotation(enableRepository = true)
    @ApiOperation("已经返回Result不做处理")
    @GetMapping("result/{userNo}")
    public Result<User> getByUserNo2(@PathVariable String userNo) {
        User user = userService.getByUserNo(userNo);
        return Result.ok(user);
    }
}

```


### RequestBody扩展

#### 使用说明
请求Body扩展类, 配合 {@link com.lemon.framework.web.annotation.RequestHeader} 支持解析Header中的参数

#### 示例说明

```java
@Data
public class RequestBodyExtendDto {
    @RequestHeader("APP")
    String app;

    String name;
}


.....省略中间代码......
  
@PostMapping
public RequestBodyExtendDto request(@RequestBody RequestBodyExtendDto extendDto){
  return extendDto;
}

```
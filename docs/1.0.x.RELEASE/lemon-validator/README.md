## 简介

`lemon-validator`
是基于hibernate-validator扩展的校验工具集，在开发中经常需要写一些字段校验的代码，比如字段非空，字段长度限制，邮箱格式验证等等，写这些与业务逻辑关系存在验证代码繁琐，重复劳动、方法内代码显得冗长等弊端，在此基础上，lemon-validator提供了以下扩展点：

- 更丰富的校验注解
- 多字段联合逻辑校验
- 资源请求完成自动校验
- 校验结果统一对象返回
- 扩展统一校验过滤器

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
    <artifactId>lemon-validator-spring-boot-starter</artifactId>
</dependency>
```

## 扩展配置

```yaml
# Config
  lemon:
    # 校验相关
    validator:
      fail-fast: true # 参数校验快速失败返回
      aspect:
        enable: true # 请求参数切面拦截
```



## 请求参数校验

springboot使用hibernate validator校验的使用在此章节不做过多的阐述，可参看以下文章

- [springboot使用hibernate validator校验](https://www.cnblogs.com/mr-yang-localhost/p/7812038.html)
- [JAVA中通过Hibernate-Validation进行参数验证](https://www.cnblogs.com/atai/p/6943404.html)

💡小提示：在[lemon-sample-quickstart](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples/-/blob/master/lemon-sample-quickstart/src/main/java/com/lemon/sample/quickstart/resource/ValidatorResource.java)
项目中有很全面的validate使用示例


### 扩展注解

扩展校验注解在`com.lemon.framework.validator.annotaion.validation`包下：

| 注解                   | 注解描述                                 |
|----------------------|--------------------------------------|
| @ApiEnumValue        | 枚举类型校验                               |
| @Chinese             | 该字段的值必须为中文                           |
| @Conditional         | 条件匹配校验规则                             |
| @Conditionals        | 多条件校验匹配规则                            |
| @DateFormat          | 日期格式校验                               |
| @IdentityNo          | 身份证件校验                               |
| @IP                  | IP格式校验                               |
| @Json                | JSON格式校验                             |
| @Mobile              | 手机号码校验                               |
| @NotAllEmpty         | 多个字段元素至少有一项值不能为空, 支持数组,对象,Map,String |
| @Num                 | 整数校验                                 |
| @Password            | 密码校验, 密码长度必须满足6~20位                  |
| @PositiveDigits      | 数值为正数, 并且数值的整数长度以及小数点长度进行校验          |
| @PositiveOrZeroDigits | 数值为正数(或者为负数), 并且数值的整数长度以及小数点长度进行校验   |
| @Split               | 分隔字符串校验                              |
| @Telephone           | 电话号码格式校验                             |
| @TelephoneOrMobile   | 允许输入电话号码或手机号码                        |
| @CompareField        | 比较字段，多个字段比较可以使用List                  |

### 参数校验

资源请求一般是 对象方式、基础类型 方式进行接收，以下对这两种参数接收方式进行说明：

- 对象方式接收：

  服务资源到服务端时，框架通过切面的方式对请求资源进行校验拦截，需要注意的是，拦截对象必须要继承`com.lemon.framework.api.data.BaseDataObject`
  类，否则不对自动校验。代码示例如下：

  ```java
  //资源处理
  @RestController
  @RequestMapping("validator")
  public class ValidatorResource {
  
      @PostMapping
      public Result valid(@RequestBody ValidatorDto dto) {
          return Result.ok();
      }
  }
  
  //请求对象
  public class ValidatorDto extends DataObjectBase {
   	.... 
  }
  
  ```

- 基础类型方式接收

  资源接收参数使用基础类型接收，对参数进行校验操作，必须在类上添加`@Validated`注解，代码示例如下

  ```java
  //资源处理
  //必须添加@Validated注解
  @Validated
  @RestController
  @RequestMapping("validator")
  public class ValidatorResource {
  		
    	//在接收参数前添加@NotBlank注解
      @GetMapping
      public Result valid(@RequestParam @NotBlank String username){
          return Result.ok();
      }
  }
  ```



### 多字段联合校验

在业务需求开发过程中，难免会遇到需要对比字段的值对比，比两次输入密码必须相同，结束时间必须大于开始时间等场景，以及自定义校验方法的应用等。以下针对此场景提供优雅的校验方法

#### 方式一：基于内置的@ScriptAssert实现

@ScriptAssert是HV内置的一个非常强大的、可以用于类级别验证注解，支持写脚本来完成验证逻辑。提供了个性化检验参数的能力。

```java
import org.hibernate.validator.constraints.ScriptAssert;

//单脚本执行
//@ScriptAssert(lang = "javascript", script = "_this.start <= _this.end", message = "start must be less than end")

//多脚本执行
@ScriptAssert.List({
        @ScriptAssert(lang = "javascript", script = "_this.start <= _this.end", message = "start must be less than end"),
        @ScriptAssert(lang = "javascript", script = "_this.startDate.isBefore(_this.endDate)", message = "startDate must be less than endDate"),
        @ScriptAssert(lang = "javascript", script = "com.lemon.sample.quickstart.utils.ValidUtils.isChineseName(_this.chineseName)", message = "名字必须是中文"),
})
@Data
public class ValidatorDto extends BaseDataObject {
    @NotNull
    Long start;

    @NotNull
    Long end;

    @NotNull
    LocalDate startDate;

    @NotNull
    LocalDate endDate;

    @NotBlank
    String chineseName;
}
```

> ⚠️ @ScriptAssert对null值并不免疫，不管咋样它都会执行的，因此书写脚本时注意判空处理


#### 方式二：结合@AssertTrue、@AssertFalse实现

使用HV字段属性级别约束特性，结合@AssertTrue、@AssertFalse校验注解，在Bean定义getter方法判断是否符合预期的值

```java
@Data
public class ValidatorDto extends BaseDataObject {

    @NotNull
    Long start2;

    @NotNull
    Long end2;

    //新增一个get方法，判断预期的值
    @AssertTrue(message = "start2 must be less than end2")
    public boolean getStar2tLessEnd2Valid() {
        return end2 >= start2;
    }
}
```


#### 方式三：使用自定义注解@CompareField比较字段大小

自定义注解@CompareField通常用于比较两个字段的值大小。

使用方法：创建一个带有@CompareField注解的自定义注解类，并标记需要比较的字段.

如下：是多组字段比较示例，使用`type`用于比较的类型，`message`为自定义返回信息

```java
@CompareField.List({
        @CompareField(first = "startDate",second = "endDate",type = CompareTypeEnum.GE,message = "开始时间应小于等于结束时间"),
        @CompareField(first = "start",second = "end")
})
@Data
public class ValidatorDto extends BaseDataObject {
  @NotNull
  Long start;

  @NotNull
  Long end;

  @NotNull
  LocalDate startDate;

  @NotNull
  LocalDate endDate;

}
```


这三种方式都可以实现类级别的验证，它俩可以说各有优劣，主要体现在如下方面：

- 方法一：@ScriptAssert是内置就提供的，因此使用起来非常的方便和通用。但缺点也是因为过于通用，因此语义上不够明显，需要阅读脚本才知。推荐少量（非重复使用）、逻辑较为简单时使用

- 方法二：通过字段属性级别约束特性，自定义getter方法判断预期值，代码稍显麻烦，但它的优点就是语义明确，灵活且不易出错，即使是复杂的验证逻辑也能轻松搞定

- 方法三：通过自定义注解@CompareField，用于简单的字段比较，代码编写简洁，语义明确且灵活，缺点就是的不能实现复杂的逻辑

总之，若你只用于比较字段大小建议使用@CompareField，若你的验证逻辑只用一次（只一个地方使用）且简单（比如只是简单判断而已），推荐使用@ScriptAssert更为轻巧。否则，你懂的~

- [文档参考](https://www.cnblogs.com/yourbatman/p/14011063.html)


### 校验结果消息

框架会捕获Validation校验异常，对校验异常信息进行转换，最终返回给调用端。框架对校验注解的message规范了统一的消息内容，如果无特殊要求下，无需执行校验的message内容。框架默认消息内容如下：

> 💡 fieldName取自Swagger的@ApiModelProperty注解内容

```properties
javax.validation.constraints.AssertFalse.message={fieldName}只能为false
javax.validation.constraints.AssertTrue.message={fieldName}只能为true
javax.validation.constraints.DecimalMax.message={fieldName}必须小于或等于{value}
javax.validation.constraints.DecimalMin.message={fieldName}必须大于或等于{value}
javax.validation.constraints.Digits.message={fieldName}数字的值超出了允许范围(只允许在{integer}位整数和{fraction}位小数范围内)
javax.validation.constraints.Email.message={fieldName}不是一个合法的电子邮件地址
javax.validation.constraints.Future.message={fieldName}需要是一个将来的时间
javax.validation.constraints.FutureOrPresent.message={fieldName}需要是一个将来或现在的时间
javax.validation.constraints.Max.message={fieldName}最大不能超过{value}
javax.validation.constraints.Min.message={fieldName}最小不能小于{value}
javax.validation.constraints.Negative.message={fieldName}必须是负数
javax.validation.constraints.NegativeOrZero.message={fieldName}必须是负数或零
javax.validation.constraints.NotBlank.message={fieldName}不能为空
javax.validation.constraints.NotEmpty.message={fieldName}不能为空
javax.validation.constraints.NotNull.message={fieldName}不能为null
javax.validation.constraints.Null.message={fieldName}必须为null
javax.validation.constraints.Past.message={fieldName}需要是一个过去的时间
javax.validation.constraints.PastOrPresent.message={fieldName}需要是一个过去或现在的时间
javax.validation.constraints.Pattern.message={fieldName}需要匹配正则表达式"{regexp}"
javax.validation.constraints.Positive.message={fieldName}必须是正数
javax.validation.constraints.PositiveOrZero.message={fieldName}必须是正数或零
javax.validation.constraints.Size.message={fieldName}个数必须在{min}和{max}之间
org.hibernate.validator.constraints.CreditCardNumber.message={fieldName}不是一个合法的信用卡号码
org.hibernate.validator.constraints.Currency.message={fieldName}不是一个合法的货币 (必须是{value}其中之一)
org.hibernate.validator.constraints.EAN.message={fieldName}不是一个合法的{type}条形码
org.hibernate.validator.constraints.Email.message={fieldName}不是一个合法的电子邮件地址
org.hibernate.validator.constraints.ISBN.message=无效的ISBN
org.hibernate.validator.constraints.Length.message={fieldName}长度需要在{min}和{max}之间
org.hibernate.validator.constraints.CodePointLength.message={fieldName}长度需要在{min}和{max}之间
org.hibernate.validator.constraints.LuhnCheck.message={fieldName} ${validatedValue}的校验码不合法, Luhn模10校验和不匹配
org.hibernate.validator.constraints.Mod10Check.message={fieldName} ${validatedValue}的校验码不合法, 模10校验和不匹配
org.hibernate.validator.constraints.Mod11Check.message={fieldName} ${validatedValue}的校验码不合法, 模11校验和不匹配
org.hibernate.validator.constraints.ModCheck.message={fieldName} ${validatedValue}的校验码不合法, ${modType}校验和不匹配
org.hibernate.validator.constraints.NotBlank.message={fieldName}不能为空
org.hibernate.validator.constraints.NotEmpty.message={fieldName}不能为空
org.hibernate.validator.constraints.ParametersScriptAssert.message={fieldName}执行脚本表达式"{script}"没有返回期望结果
org.hibernate.validator.constraints.Range.message={fieldName}需要在{min}和{max}之间
org.hibernate.validator.constraints.SafeHtml.message={fieldName}可能有不安全的HTML内容
org.hibernate.validator.constraints.ScriptAssert.message={fieldName}执行脚本表达式"{script}"没有返回期望结果
org.hibernate.validator.constraints.UniqueElements.message={fieldName}必须只包含唯一的元素
org.hibernate.validator.constraints.URL.message={fieldName}需要是一个合法的URL
org.hibernate.validator.constraints.time.DurationMax.message={fieldName}必须小于${inclusive == true ? '或等于' : ''}${days == 0 ? '' : days += '天'}${hours == 0 ? '' : hours += '小时'}${minutes == 0 ? '' : minutes += '分钟'}${seconds == 0 ? '' : seconds += '秒'}${millis == 0 ? '' : millis += '毫秒'}${nanos == 0 ? '' : nanos += '纳秒'}
org.hibernate.validator.constraints.time.DurationMin.message={fieldName}必须大于${inclusive == true ? '或等于' : ''}${days == 0 ? '' : days += '天'}${hours == 0 ? '' : hours += '小时'}${minutes == 0 ? '' : minutes += '分钟'}${seconds == 0 ? '' : seconds += '秒'}${millis == 0 ? '' : millis += '毫秒'}${nanos == 0 ? '' : nanos += '纳秒'}
custom.validation.constraints.ApiEnumValue.message={fieldName}格式错误
custom.validation.constraints.Chinese.message={fieldName}必须是中文
custom.validation.constraints.Alphanumeric.message={fieldName}必须是字母或数字
custom.validation.constraints.IdentityNo.message={fieldName}不是一个合法的证件号码
custom.validation.constraints.IP.message={fieldName}不是一个合法的IP地址
custom.validation.constraints.Json.message={fieldName}不是一个合法的JSON格式
custom.validation.constraints.Mobile.message={fieldName}不是一个合法的手机号码
custom.validation.constraints.Password.message={fieldName}不是一个合法的密码, 长度需要在6到20之间
custom.validation.constraints.Split.message={fieldName}格式错误(分割符必须为[{separator}], 且分隔后元素数量必须在{minSize}和{maxSize}之间)
custom.validation.constraints.Telephone.message={fieldName}不是一个合法的电话号码
custom.validation.constraints.Num.message={fieldName}必须是一个整数
custom.validation.constraints.TelephoneOrMobile.message={fieldName}不是一个合法的手机号码或电话号码
custom.validation.constraints.DateFormat.message={fieldName}不是一个有效的日期格式, 格式必须为 {format}
custom.validation.constraints.PositiveDigits.message={fieldName}必须为正数并且数字的值只允许在{integer}位整数和{fraction}位小数范围内
custom.validation.constraints.PositiveOrZeroDigits.message={fieldName}必须为正数或者为0并且数字的值只允许在{integer}位整数和{fraction}位小数范围内
custom.validation.constraints.SpecificValueSplit.message={fieldName}传入的参数内容不合法, 仅允许传入{allowValue}, 且分隔后元素数量必须在{minSize}和{maxSize}之间
custom.validation.constraints.compare.message={first} 应小于 {second}
```

### 知识补充
- [spring SpEL表达式---基础接口及基础语法](https://blog.csdn.net/drxRose/article/details/85085716)


## 校验过滤器

### 如何使用

编写业务代码过程中，往往需要编写许多Check前置逻辑，并且与实际业务编码进行混编，导致代码可读性差，并且校验逻辑存在重复编码的情况，后期不方便维护。框架包提供`@ValidatorFilters`
注解进行统一处理, 规范校验逻辑;

💡小提示：在[lemon-sample-quickstart](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples/-/tree/master/lemon-sample-quickstart/src/main/java/com/lemon/sample/quickstart/validation)
项目中有很全面的validate使用示例

### 扩展注解

扩展校验注解在`com.lemon.framework.validator.annotaion`包下：

| 注解                | 注解描述             |
|-------------------|------------------|
| @ValidatorFilters | 校验过滤器注解, 支持多个校验器 |
| @ParamValue       | 注入请求参数的对象或属性     |
| @HeaderValue      | 注入HTTP HEADER中的值 |

### 使用示例

```java
//定义校验器代码如下: 实现com.lemon.framework.validator.ValidatorFilter类
@Slf4j
//必须申明为多例模式, 否则并发场景下会导致@ParamValue,@HeaderValue幅值被覆盖情况
@Scope("prototype")
@Component
public class TestValidator extends ValidatorFilter {
    @ParamValue
    String merchNo;

    @Override
    protected Result process(Map<String, Object> parameterMap) {
        log.info(this.merchNo);
    }
}

//====================================================================================
//使用示例1: 请求方法基础类型属性方式访问
@Component
public class TestService {
    @ValidatorFilters(filter = TestValidator.class)
    public void handle(String merchNo) {
        //....
    }
}

//====================================================================================
//使用示例2: 对象方式访问注入参数
@Component
public class TestService {
    @ValidatorFilters(filter = TestValidator.class)
    public void handle(TestModel model) {
        //....
    }
}

public class TestModel {
    String merchNo;
    //get set method ...
}

//====================================================================================
//使用示例3: 将请求参数对象注入到校验器中,
//注意: Model必须继承 {@link com.lemon.framework.api.data.BaseDataObject}
//校验器代码如下:
@Slf4j
//必须申明为多例模式, 否则并发场景下会导致@ParamValue,@HeaderValue幅值被覆盖情况
@Scope("prototype")
@Component
public class TestValidator extends ValidatorFilter {
    @ParamValue
    TestModel model;

    @Override
    protected Result process(Map<String, Object> parameterMap) {
        log.info(model.getMerchNo());
    }
}

@Component
public class TestService {
    @ValidatorFilters(filter = TestValidator.class)
    public void handle(TestModel model) {
        //....
    }
}

public class TestModel extends DataObjectBase {
    String merchNo;
    //get set method ...
}
```
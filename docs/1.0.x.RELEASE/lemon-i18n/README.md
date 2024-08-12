## 简介

`lemon-i18n`是基于spring Boot实现的国际化的工具包，扩展了国际化资源配置的支持，主要扩展以下功能：

- 根据用户语言偏好，响应消息内容、定义枚举文本支持多语言；
- 消息资源支持多文件夹，方便维护管理；
- 支持服务端从其他途径拉取资源信息；

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
    <artifactId>lemon-i18n-spring-boot-starter</artifactId>
</dependency>
```

## 消息国际化

### 扩展配置

```yml
# Config
  lemon:
    # 消息国际化
    i18n:
      enable: true # 国际化消息资源开关
      base-folder: i18n # 资源文件目录, 默认 i18n
```

### 语言切换

> 框架约定在Headers中添加`Accept-Language`属性, 设置语言内容，可选值为国际化国标,
> 参考[《每个国家对应的语言Locale和国家代码对照表》](https://blog.csdn.net/zhangatm/article/details/84108338)
>
> ⚠️系统默认的语言为zh_CN

![image-20191206160312923](_media/image-20191206160312923.png ':size=60%')

### 资源配置方式

> 以下两种方式可以共存，可根据实际情况选择任意方式集成

#### 方式一：properties配置

在`resources`目录下新建`i18n`文件夹，(支持对资源分多目录存放)，新建对应语言文件如下:


![image-20191206155746388](_media/image-20191206155746388.png ':size=60%')

#### 方式二：获取远端资源

> 资源通过远端方式，需要实现`com.lemon.framework.i18n.MessageResourceProvide`接口即可, 接口定义如下：

```java
public interface MessageResourceProvide {
    /**
     * 加载国际化资源信息
     *
     * @return
     */
    List<Resource> loadMessageResource();
}
```

### 代码示例

```java

@RequestMapping("i18n")
@RestController
public class I18nResource {

    @GetMapping("source")
    public Result source() {
        return Result.fail("unknown.error");
    }


    @GetMapping("properties")
    public Result properties() {
        return Result.fail("unknown.error");
    }
}
```

> demo已经在`lemon-sample-quickstart`项目中有实现, 感兴趣的朋友可以去看看, 效果如下:

1. 使用中文语言

![image-20191118200923727](_media/image-20191118200923727.png ':size=60%')

2. 使用英文语言

![image-20191118200938590](_media/image-20191118200938590.png ':size=60%')


## 枚举国际化

为接口返回的枚举类型数据提供国际化支持，使得不同语言环境下能够正确显示对应的文本。
> 例如，一个订单状态可以包括"待支付"、"已支付"和"已取消"等几个选项。当接口返回这样的枚举类型数据时，通常会以数字或英文字符串的形式进行表示，比如1代表"待支付"，2代表"已支付"等。 然而，在多语言环境下，需要将这些枚举值对应的文本进行翻译，以便用户能够理解和使用，多端无需再关注枚举定义，由后端接口统一处理返回。

### 响应枚举属性

代码片段如下：
```java
@Data
public class UserRespVo extends BaseDataObject {
    private String userNo;
    private UserTypeEum userType;
    ....
```

### 枚举定义

⚠️枚举必须实现`com.lemon.framework.api.ApiEnum`接口，

- 配置方式：

1. 默认根据类名.枚举name动态获取i18n定义资源KEY
2. 通过`text`参数定义国际化Key

```java
public enum UserTypeEum implements ApiEnum<String> {
    ADMIN("系统管理员"),
    USER("user.type.user");

    String text;
    UserTypeEum(String text) {
        this.text = text;
    }
    @Override
    public String getValue() {
        return this.name();
    }
    @Override
    public String getText() {
        return this.text;
    }
}
```

以上示例代码详见: [GIT](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples/-/blob/master/lemon-sample-quickstart/src/main/java/com/lemon/sample/quickstart/enums/UserTypeEum.java?ref_type=heads)

### 国际化配置

与上述消息国际化配置一致，在`resources/i18n`目录下配置枚举`text`国际化编码

> 枚举国际化配置建议与消息国际化分配置分开管理，防止后续不方便维护。

![img.png](_media/enum.png ':size=60%')


### 响应结果

根据用户语言设置，从相应的资源文件中读取对应的文本进行显示。

![img.png](_media/enum_result.png ':size=60%')

与`Web组件`处理枚举序列化一致，额外返回`Convert`字段显示国际化枚举`Text`内容
```json
{
  "userType": "ADMIN",
  "userTypeConvert": {
    "format": "admin"
  }
}
```


## 知识补充

> 系统默认国际化配置表如下：
>
> 如果不满足系统要求，可以自行对错误消息编码进行重新配置！

| 错误消息编码 | 错误描述      | 场景说明                                 | 系统默认消息         |
|--------|-----------|--------------------------------------|----------------|
| 400    | 错误的请求     | 请求包含请求参数语法错误，请求Method不正确、枚举值不符合规范等错误 | 错误的请求 {}       |
| 405    | 非法的操作请求{} | 请求资源出现非正常或不满足条件时的操作异常                | 非法的操作请求 {}     |
| 406    | 查无结果      | 获取资源所需要的策略并没有被满足                     | {}查无结果         |
| 408    | 请求超时      | 网关服务器尝试执行请求时，未能及时从上游服务器              | 网络开小差了，请重试     |
| 409    | 数据已存在     | 由于数据已经存在，服务器无法存储完成请求的内容              | {}数据已存在        |
| 412    | 请求参数错误    | 一般是用户传入参数非法引起的，请仔细检查入参格式、范围是否一一对应    | 请求参数错误:{}      |
| 429    | 请求限流      | 服务器达到请求流量限制，拒绝请求访问                   | 系统繁忙，请稍后再试     |
| 500    | 系统未知错误    | 多数是由未知异常引起的，仔细检查传入的参数是否符合文档描述        | 系统异常 {}        |
| 502    | 调用远程服务异常  | 调用其他第三方(feign)服务发生错误                 | 上游服务器收到无效响应    |
| 503    | 操作频繁错误    | 重复请求某资源导致操作频繁错误                      | 您的操作过于频繁，请稍后再试 |

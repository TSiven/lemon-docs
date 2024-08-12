## 简介

​    `lemon-framework`是基于`Spring Cloud`微服务化的`开发平台`，作为后端服务的开发基础框架集成。代码简洁，架构清晰。
核心技术采用`Spring Boot 2.6.13`、`Spring Cloud (2021.0.5) `以及`Spring Cloud Alibaba(2021.0.5.0)`相关核心组件，提供了`autoconfigure`
自动装配的能力，开箱即用，项目可根据依赖需要灵活引用`pom`模块。

![framework.png](_media/lemon-framework.png ':size=60%')

## 核心依赖

| 依赖                                           | 版本         |
|----------------------------------------------|------------|
| Spring Boot                                  | 2.6.13     |
| Spring Cloud                                 | 2021.0.5   |
| Spring Cloud Alibaba                         | 2021.0.5.0 |
| Mybatis                                      | 3.5.13     |
| [hutool-core](https://hutool.cn/)            | 5.8.16     |
| [xxl-job](http://www.xuxueli.com/xxl-job/#/) | 2.4.0      |
| lombok                                       | 1.18.22    |
| fastjson2                                    | 2.0.38     |
| easyexcel                                    | 3.3.2      |
| ...                                          |            |

## 模块说明

### Component

​    `Component`维护公共的组件代码，它帮助我们简化每一行代码，减少每一个方法，后续慢慢积累会持续并加入更多非业务相关功能，最终形成丰富的工具集。

| 模块名称            | 模块说明                     |
|-----------------|--------------------------|
| lemon-dependencies | 父pom为应用程序提供依赖关系和插件管理     |
| lemon-api          | 公共API支持                  |
| lemon-cache        | 缓存组件支持                   |
| lemon-database     | 数据库相关支持                  |
| lemon-exception    | 异常组件支持                   |
| lemon-feign        | spring cloud feign调用组件支持 |
| lemon-i18n         | 国际化组件支持                  |
| lemon-logger       | 日志组件支持                   |
| lemon-utils        | 常用工具集                    |
| lemon-validator    | 校验相关支持                   |
| lemon-web          | WEB相关支持                  |
| ...             |                          |

### Starter

​
使用SpringBoot快速的开发基于Spring框架的项目。围绕SpringBoot存在很多开箱即用的Starter依赖，同时根据业务开发积累，后续我们亦可制定更加灵活便利的starter，使得我们在开发业务代码时能够非常方便的、不需要过多关注框架的配置，而只需要关注业务即可。

| 模块名称                             | 模块说明                                                         |
|----------------------------------|--------------------------------------------------------------|
| lemon-spring-boot-starter           | Spring boot启动器，该项目主要利用Spring Boot的自动化配置特性来实现快速对`framework`进行整合 |
| lemon-spring-boot-autoconfigure     | Spring boot自动装配，提供默认的配置，以及YMAL的配置，开箱即用                       |
| lemon-cache-spring-boot-starter     | 缓存spring boot启动器                                             |
| lemon-database-spring-boot-starter  | 数据库spring boot启动器                                            |
| lemon-feign-spring-boot-starter     | feign spring boot启动器                                         |
| lemon-i18n-spring-boot-starter      | 国际化 spring boot启动器                                           |
| lemon-validator-spring-boot-starter | 校验spring boot启动器                                             |
| lemon-web-spring-boot-starter       | WEB spring boot启动器                                           |
| lemon-all-spring-boot-starter      | 集成基础的spring cloud组件、以及组件启动器， 开箱即用                            |
| ...                              |                                                              |

## 快速开始

### 编译

```shell
cd `lemon-framework dir`
mvn clean package install
```

### 安装

> **示例项目** 可参考项目下的[lemon-samples](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples)
> 目录。

1. 在项目的`pom.xml`中添加`parent`父依赖

```xml
<parent>
    <groupId>com.lemon.framework</groupId>
    <artifactId>lemon-dependencies</artifactId>
    <version>最新版本</version>
</parent>
```

2. 在项目`pom.xml`的`dependencies`中加入以下内容：

> 可以根据需求对每个模块单独引入，也可以通过引入`lemon-all-spring-boot-starter`方式引入所有spring cloud相关模块。

```xml
<dependency>
    <groupId>com.lemon.framework</groupId>
    <artifactId>lemon-all-spring-boot-starter</artifactId>
</dependency>
```

# 如何维护

## component

- 命名规范：
    - 前缀：lemon
    - 模式：lemon-模块名
    - 举例：lemon-api、lemon-validator、lemon-logger

## 如何自定义一个Starter

### 启动器(starter)

- 启动器模块是一个空的JAR文件，仅提供辅助性依赖管理，这些依赖可能用于自动配置或者其他类库

- 命名规范：

  > 推荐使用以下命名规范:

    - 官方命名空间
        - 前缀：spring-boot-starter
        - 模式：spring-boot-starter-模块名
        - 举例：spring-boot-starter-web、spring-boot-starter-jdbc、spring-boot-starter-actuator
    - 自定义命名空间
        - 前缀：lemon-spring-boot-starter / lemon-spring-cloud-starter
        - 模式：lemon-spring-boot-starter-模块名 / lemon-spring-cloud-starter-模块名
        -
      举例：lemon-spring-boot-starter、lemon-spring-boot-autoconfigure、lemon-all-spring-boot-starter、lemon-spring-cloud-starter-basic

### 参考文章

- [编写自己的SpringBoot-starter](https://www.cnblogs.com/yuansc/p/9088212.html)

## 如何编写自动配置

> 通过`lemon-spring-boot-autoconfigure`项目模块完成自动化配置。

### 自动化配置需要的注解配置

```java
@Configuration //指定这个类是一个配置类

@ConfigurationPropertie //结合相关xxxProperties类来绑定相关的配置
@EnableConfigurationProperties //这是一个开启使用配置参数的注解，value值就是我们配置实体参数映射的ClassType，将配置实体作为配置来源。

@AutoConfigureAfter //指定自动配置类的顺序

@Bean //给容器中添加组件

@ConditionalOnBean //当SpringIoc容器内存在指定Bean的条件
@ConditionalOnClass //当SpringIoc容器内存在指定Class的条件
@ConditionalOnExpression //基于SpEL表达式作为判断条件
@ConditionalOnJava //基于JVM版本作为判断条件
@ConditionalOnJndi //在JNDI存在时查找指定的位置
@ConditionalOnMissingBean //当SpringIoc容器内不存在指定Bean的条件
@ConditionalOnMissingClass //当SpringIoc容器内不存在指定Class的条件
@ConditionalOnNotWebApplication //当前项目不是Web项目的条件
@ConditionalOnProperty //指定的属性是否有指定的值
@ConditionalOnResource //类路径是否有指定的值
@ConditionalOnSingleCandidate //当指定Bean在SpringIoc容器内只有一个，或者虽然有多个但是指定首选的Bean
@ConditionalOnWebApplication //当前项目是Web项目的条件
```

### 自动配置类加载

将需要启动就加载的自动配置类，配置在META-INF/spring.factories

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.lemon.framework.autoconfigure.startup.StartupAutoConfiguration,\
  com.lemon.framework.autoconfigure.exception.ExceptionAutoConfiguration,\
  com.lemon.framework.autoconfigure.validator.ValidatorAutoConfiguration,\
  com.lemon.framework.autoconfigure.job.JobAutoConfiguration,\
  com.lemon.framework.autoconfigure.feign.FeignAutoConfiguration,\
  com.lemon.framework.autoconfigure.cache.CacheAutoConfiguration,\
  com.lemon.framework.autoconfigure.context.SpringContextAutoConfiguration,\
  com.lemon.framework.autoconfigure.web.WebAutoConfiguration,\
  com.lemon.framework.autoconfigure.retry.SpringRetryAutoConfiguration,\
  com.lemon.framework.autoconfigure.async.AsyncAutoConfiguration,\
  com.lemon.framework.autoconfigure.event.EventAutoConfiguration,\
  com.lemon.framework.autoconfigure.i18n.I18nAutoConfiguration
  ...

org.springframework.boot.env.EnvironmentPostProcessor=\
  com.lemon.framework.autoconfigure.env.CustomEnvironmentPostProcessor
```

### 参考文章

- [SpringBoot使用AutoConfiguration自定义Starter](https://segmentfault.com/a/1190000011433487)

# 更新记录

[CHANGELOG](/CHANGELOG.md)
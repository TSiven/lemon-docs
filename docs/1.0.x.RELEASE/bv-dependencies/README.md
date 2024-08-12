## 简介

> 框架父pom为应用程序提供依赖关系和插件管理

![lemon-framework.png](../../_media/lemon-framework.png ':size=60%')

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
| mysql-connector-java                         | 8.0.11     |
| guava                                        | 27.1-jre   |
| commons-lang3                                | 3.7        |
| ...                                          |            |

## 安装

> **示例项目** 可参考项目下的[lemon-samples](https://codeup.aliyun.com/66b98e63c3f44f74f4310473/framework/lemon-samples)
> 目录。

1. 在项目的`pom.xml`中添加`parent`父依赖

```xml
<parent>
    <groupId>com.lemon.framework</groupId>
    <artifactId>lemon-dependencies</artifactId>
    <version>1.1.3.RELEASE</version>
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
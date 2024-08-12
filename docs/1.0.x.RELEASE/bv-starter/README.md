
## Starter

> ​ 使用SpringBoot快速的开发基于Spring框架的项目。围绕SpringBoot存在很多开箱即用的Starter依赖，同时根据业务开发积累，后续我们亦可制定更加灵活便利的starter，使得我们在开发业务代码时能够非常方便的、不需要过多关注框架的配置，而只需要关注业务即可。

| <span style="white-space:nowrap;">模块名称&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;</span> | 模块说明                                                                  |
|-----------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| lemon-spring-boot-starter                                                                                                                  | Spring boot启动器，该项目主要利用Spring Boot的自动化配置特性来实现快速对`framework`进行整合        |
| lemon-spring-boot-autoconfigure                                                                                                            | Spring boot自动装配，提供默认的配置，以及YMAL的配置，开箱即用                                |
| lemon-cache-spring-boot-starter                                                                                                            | 缓存spring boot启动器                                                      |
| lemon-database-spring-boot-starter                                                                                                         | 数据库spring boot启动器                                                     |
| lemon-feign-spring-boot-starter                                                                                                            | feign spring boot启动器                                                  |
| lemon-i18n-spring-boot-starter                                                                                                             | 消息国际化spring boot启动器                                                   |
| lemon-logger-spring-boot-starter                                                                                                           | 日志spring boot启动器                                                      |
| lemon-validator-spring-boot-starter                                                                                                        | 参数校验spring boot启动器                                                    |
| lemon-web-spring-boot-starter                                                                                                              | WEB spring boot启动器                                                    |
| lemon-all-spring-boot-starter                                                                                                             | 框架全家桶，集成了Spring cloud相关依赖以及framework所有`lemon-spring-boot-starter`启动器的依赖。 |
| ...                                                                                                                                     |                                                                       |

## Properties

> 框架自动装配`lemon-spring-boot-autoconfigure`提供了YMAL扩展的能力，详见[配置](../config.md)


## Package Scan
> 自动装配默认扫描包路径

### Mapper
```
com.lemon.**.mapper
```

### Feign
```
com.lemon.**.feign
```
```yaml
# Config
lemon:
  # 日志配置
  logging:
    # 总开关, 为false将关闭日志功能, 默认开启
    enable: true
    # 输出日志内容级别, 默认：BASIC
    level: full
    # 日志打印最大字符长度（请求、响应参数），默认400个字符, 配置小于等于0不进行截取
    maximum-log-length: 400
    # 开启保存日志存储
    enable-repository: true
    # 系统全局默认日志存储器
    global-logging-repository: com.lemon.framework.logger.DefaultLoggingRepository
    
  # 校验相关
  validator:
    fail-fast: true # 参数校验快速失败返回
    aspect:
      enable: true # 请求参数切面拦截
  job:
    enable: false
    admin-addresses: http://xxl-job.com:8080/xxl-job-admin # 调度中心部署跟地址
    app-name: lemon-agent-task # 执行器AppName
    access-token: TestJobSign1024
    port: 9098 # 执行器端口号
    log-path: ./logs/jobhandler # 执行器运行日志文件存储磁盘路径
    log-retention-days: 7 # 执行器日志保存天数, 默认7天
  
  # 缓存管理
  cache:
    manager:
      enable: true
      default-expire-type: hours
      default-expire-time: 1
  
  # 消息国际化
  i18n:
    enable: true
    base-folder: i18n
  
  # WEB配置
  web:
    # 序列化配置 `1.0.6.RELEASE` 以上版本支持
    serialization:
      # 是否包含null值属性。不包含(NON_NULL)，包含（always）
      include: non_null
      domain: https://static.bivbris.com # 动态替换域名资源 `1.0.0.RELEASE` 以上版本支持
      enable-sensitive: true # 是否开启脱敏 `1.0.0.RELEASE` 以上版本支持
    # 日期格式化配置
    date-format:
      serializer-to-user-time-zone: true # 序列化：系统时区TO 用户时区
      deserializer-to-system-time-zone: true # 反序列化：用户时区TO 系统时区
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
  
  # 异常配置 `1.0.1.RELEASE` 以上版本支持
  exception:
    enable-global-advice: true
```
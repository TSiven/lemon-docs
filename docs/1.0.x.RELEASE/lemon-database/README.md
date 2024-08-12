## 简介

`lemon-database`是对基于主要扩展如下：
- 封装数据分页工具类

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
    <artifactId>lemon-database-spring-boot-starter</artifactId>
</dependency>
```

## 如何使用
### 分页工具类

分页工具类主要提供以下APIs
- 源代码：com.lemon.framework.database.PageUtil

```java
public class PageUtil {

    public static <T> ResultPage<T> convertResult(PageInfo<T> page) {
        return ResultPage.success(page.getList(), page.getTotal(), page.getPageNum(), page.getPageSize());
    }

    public static <T, C> ResultPage<C> convertResult(PageInfo<T> page, Function<T, C> convert) {
        List<T> list = page.getList();
        List<C> convertResults = list.stream().map(m -> convert.apply(m)).collect(Collectors.toList());
        return ResultPage.success(convertResults, page.getTotal(), page.getPageNum(), page.getPageSize());
    }


    /**
     * 分页查询封装
     *
     * @param buildQuery 构建查询条件
     * @param execute    执行查询逻辑
     * @param <T>        查询条件对象
     * @param <R>        执行查询逻辑返回对象
     * @return
     */
    public static <T extends PageVo, R> PageInfo<R> pageQuery(Supplier<T> buildQuery, Function<T, List<R>> execute) {
        T query = buildQuery.get();
        PageHelper.startPage(query.getPageNo(), query.getPageSize());
        List<R> listData = execute.apply(query);
        PageInfo<R> pageInfo = new PageInfo<>(listData); // 将结果封装到 PageInfo 对象中
        return pageInfo;
    }


    /**
     * 分页查询封装
     *
     * @param buildQuery 构建查询条件
     * @param execute    执行查询逻辑
     * @param convert    执行结果转换封装
     * @param <T>        查询条件对象
     * @param <R>        执行查询逻辑返回对象
     * @param <C>        转换结果对象
     * @return
     */
    public static <T extends PageVo, R, C> PageInfo<C> pageQuery(Supplier<T> buildQuery, Function<T, List<R>> execute, Function<R, C> convert) {
        PageInfo<R> pageInfo = pageQuery(buildQuery, execute);
        return convert(pageInfo, convert);
    }


    /**
     * 分页对象 <R> TO <C>
     *
     * @param pageInfo 分页结果对象
     * @param convert  执行结果转换封装
     * @param <R>      执行查询逻辑返回对象
     * @param <C>      转换结果对象
     * @return
     */
    public static <R, C> PageInfo<C> convert(PageInfo<R> pageInfo, Function<R, C> convert) {

        PageInfo<C> resultPage = new PageInfo<>();
        resultPage.setTotal(pageInfo.getTotal());
        resultPage.setPageNum(pageInfo.getPageNum());
        resultPage.setPageSize(pageInfo.getPageSize());
        resultPage.setPages(pageInfo.getPages());

        List<R> list = pageInfo.getList();
        if (list.isEmpty()) {
            resultPage.setList(new ArrayList());
            return PageInfo.emptyPageInfo();
        }

        List<C> cResult = list.stream().map(r -> convert.apply(r)).collect(Collectors.toList());
        resultPage.setList(cResult);
        return resultPage;
    }

}

```

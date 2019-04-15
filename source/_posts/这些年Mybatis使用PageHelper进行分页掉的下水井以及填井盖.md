---
title: 这些年Mybatis使用PageHelper进行分页掉的下水井以及填井盖
date: 2019-04-13 19:02:54
tags:
---

## 这些年Mybatis使用PageHelper进行分页掉的下水井以及填井盖

#### 前言

日常业务开发中，列表分页查询的需求非常常见，在Mybatis 架构中，我们又非常喜欢使用[PageHelper](https://github.com/pagehelper/Mybatis-PageHelper) 这个插件，非常感谢开源这个插件的大佬～

虽然大佬把文档写的非常全，然而依然掉下水井🤦‍♂️，以下记录一下掉的下水井，以及如何填井盖的过程。

[使用文档](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md>)一定要仔细看，填盖过程中引用了文档的相关内容。

#### 问题描述

在单元测试测试和测试环境中分别出现了两个问题，让我怀疑了人生。

单元测试问题：

查询报错，无缘无故的加了一个order by 排序，查看sql 和 方法并没有order by的逻辑，并且没有其他的逻辑(也没有PageHelper 配置)，真的活见鬼了，最见鬼的是非必现。

测试环境问题：

程序执行一段时间后，一个分页接口直接报错了，报错的内容见下图，同样是非现，代码也没有特殊的逻辑，一个查询，使用PageHelper.startPage(pageNum, pageSize)进行分页。

```
报错信息：
09:41:22.528 [http-nio-8080-exec-1] ERROR o.a.c.c.C.[.[.[.[dispatcherServlet] - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.type.TypeException: Could not set parameters for mapping: ParameterMapping{property='Second_PageHelper', mode=IN, javaType=class java.lang.Integer, jdbcType=null, numericScale=null, resultMapId='null', jdbcTypeName='null', expression='null'}. Cause: org.apache.ibatis.type.TypeException: Error setting non null for parameter #1 with JdbcType null . Try setting a different JdbcType for this parameter or a different configuration property. Cause: java.sql.SQLException: Type not supported] with root cause
java.sql.SQLException: Type not supported
at com.informix.util.IfxErrMsg.buildException(IfxErrMsg.java:480)
at com.informix.util.IfxErrMsg.getSQLException(IfxErrMsg.java:449)
at com.informix.util.IfxErrMsg.getSQLException(IfxErrMsg.java:400)
at com.informix.jdbc.IfxValue.getIfxTypeName(IfxValue.java:207)
at com.informix.jdbc.IfxValue.makeInstance(IfxValue.java:342)
at com.informix.jdbc.IfxValue.makeInstance(IfxValue.java:334)
at com.informix.jdbc.IfxPreparedStatement.setInt(IfxPreparedStatement.java:989)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.apache.tomcat.jdbc.pool.StatementFacade$StatementProxy.invoke(StatementFacade.java:114)
at com.sun.proxy.$Proxy133.setInt(Unknown Source)
at org.apache.ibatis.type.IntegerTypeHandler.setNonNullParameter(IntegerTypeHandler.java:31)
at org.apache.ibatis.type.IntegerTypeHandler.setNonNullParameter(IntegerTypeHandler.java:26)
at org.apache.ibatis.type.BaseTypeHandler.setParameter(BaseTypeHandler.java:53)
at org.apache.ibatis.scripting.defaults.DefaultParameterHandler.setParameters(DefaultParameterHandler.java:87)
at org.apache.ibatis.executor.statement.PreparedStatementHandler.parameterize(PreparedStatementHandler.java:93)
at org.apache.ibatis.executor.statement.RoutingStatementHandler.parameterize(RoutingStatementHandler.java:64)
at org.apache.ibatis.executor.SimpleExecutor.prepareStatement(SimpleExecutor.java:86)
at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:62)
at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:326)
at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:156)
at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:109)
at com.github.pagehelper.PageInterceptor.intercept(PageInterceptor.java:136)
at org.apache.ibatis.plugin.Plugin.invoke(Plugin.java:61)
at com.sun.proxy.$Proxy131.query(Unknown Source)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:148)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:141)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:433)
at com.sun.proxy.$Proxy81.selectList(Unknown Source)
at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:230)
at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:139)
at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:76)
at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:59)
at com.sun.proxy.$Proxy87.selectByExample(Unknown Source)
at com.ccb.hebei.zcyd.dao.service.impl.ContractRecordStateServiceImpl.search(ContractRecordStateServiceImpl.java:50)
at com.ccb.hebei.zcyd.web.api.RecordStateApi.search(RecordStateApi.java:42)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)
at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:133)
at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:97)
at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738)
at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:967)
at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:901)
at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970)
at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:872)
at javax.servlet.http.HttpServlet.service(HttpServlet.java:661)
at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846)
at javax.servlet.http.HttpServlet.service(HttpServlet.java:742)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99)
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
at org.springframework.web.filter.HttpPutFormContentFilter.doFilterInternal(HttpPutFormContentFilter.java:109)
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:81)
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:197)
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:198)
at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:496)
at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:140)
at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:81)
at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:87)
at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:342)
at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:803)
at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66)
at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:790)
at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1459)
at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
at java.lang.Thread.run(Thread.java:745)
```

```
代码模块：
 PageHelper.startPage(pageNum, pageSize);
 list = countryMapper.selectIf(param1);
```

#### 问题解析

欲解决问题，就要了解PageHelper是怎么实现的，并且PageHelper.startPage(pageNum, pageSize) 干了啥事。

这次现看PageHelper.startPage(pageNum, pageSize)的源码：

```

package com.github.pagehelper.page;

import com.github.pagehelper.ISelect;
import com.github.pagehelper.Page;
import com.github.pagehelper.util.PageObjectUtil;

import java.util.Properties;

/**
 * 基础分页方法
 *
 * @author liuzh
 */
public abstract class PageMethod {
    protected static final ThreadLocal<Page> LOCAL_PAGE = new ThreadLocal<Page>();
    protected static boolean DEFAULT_COUNT = true;

    /**
     * 设置 Page 参数
     *
     * @param page
     */
    protected static void setLocalPage(Page page) {
        LOCAL_PAGE.set(page);
    }

    /**
     * 获取 Page 参数
     *
     * @return
     */
    public static <T> Page<T> getLocalPage() {
        return LOCAL_PAGE.get();
    }

    /**
     * 移除本地变量
     */
    public static void clearPage() {
        LOCAL_PAGE.remove();
    }

    /**
     * 开始分页
     *
     * @param params
     */
    public static <E> Page<E> startPage(Object params) {
        Page<E> page = PageObjectUtil.getPageFromObject(params, true);
        //当已经执行过orderBy的时候
        Page<E> oldPage = getLocalPage();
        if (oldPage != null && oldPage.isOrderByOnly()) {
            page.setOrderBy(oldPage.getOrderBy());
        }
        setLocalPage(page);
        return page;
    }

    /**
     * 开始分页
     *
     * @param pageNum  页码
     * @param pageSize 每页显示数量
     */
    public static <E> Page<E> startPage(int pageNum, int pageSize) {
        return startPage(pageNum, pageSize, DEFAULT_COUNT);
    }

    /**
     * 开始分页
     *
     * @param pageNum  页码
     * @param pageSize 每页显示数量
     * @param count    是否进行count查询
     */
    public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count) {
        return startPage(pageNum, pageSize, count, null, null);
    }

    /**
     * 开始分页
     *
     * @param pageNum  页码
     * @param pageSize 每页显示数量
     * @param orderBy  排序
     */
    public static <E> Page<E> startPage(int pageNum, int pageSize, String orderBy) {
        Page<E> page = startPage(pageNum, pageSize);
        page.setOrderBy(orderBy);
        return page;
    }

    /**
     * 开始分页
     *
     * @param pageNum      页码
     * @param pageSize     每页显示数量
     * @param count        是否进行count查询
     * @param reasonable   分页合理化,null时用默认配置
     * @param pageSizeZero true且pageSize=0时返回全部结果，false时分页,null时用默认配置
     */
    public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
        Page<E> page = new Page<E>(pageNum, pageSize, count);
        page.setReasonable(reasonable);
        page.setPageSizeZero(pageSizeZero);
        //当已经执行过orderBy的时候
        Page<E> oldPage = getLocalPage();
        if (oldPage != null && oldPage.isOrderByOnly()) {
            page.setOrderBy(oldPage.getOrderBy());
        }
        setLocalPage(page);
        return page;
    }
.....忽略了一些代码........
}

```

发现PageHelper中使用ThreadLocal，ThreadLocal有什么特性呢？详情请看下篇[文章](https://zeyiy.github.io/2019/04/13/ThreadLocal%E7%9A%84%E7%90%86%E8%A7%A3/)讲解。先说结论，

结论是：分页参数和线程是绑定的。

只要你可以保证在 `PageHelper` 方法调用后紧跟 MyBatis 查询方法，这就是安全的。因为 `PageHelper` 在 `finally` 代码段中自动清除了 `ThreadLocal` 存储的对象。

如果代码在进入 `Executor` 前发生异常，就会导致线程不可用，这属于人为的 Bug（例如接口方法和 XML 中的不匹配，导致找不到 `MappedStatement` 时）， 这种情况由于线程不可用，也不会导致 `ThreadLocal` 参数被错误的使用。

不安全的代码示范：

```
PageHelper.startPage(1, 10);
List<Country> list;
if(param1 != null){
    list = countryMapper.selectIf(param1);
} else {
    list = new ArrayList<Country>();
}
```

这种情况下由于 param1 存在 null 的情况，就会导致 PageHelper 生产了一个分页参数，但是没有被消费，这个参数就会一直保留在这个线程上。当这个线程再次被使用时，就可能导致不该分页的方法去消费这个分页参数，这就产生了莫名其妙的分页。

同样orderBy同理。

查看全局代码发现，确实有这样的代码逻辑🤦‍♂️。这同时解释了为啥不是必现的原因了，因为触发未消费的分页参数的逻辑不是每次都执行。

#### 怎么填井盖呢？

1. PageHelper 紧跟查询语句

```
List<Country> list;
if(param1 != null){
    PageHelper.startPage(1, 10);
    list = countryMapper.selectIf(param1);
} else {
    list = new ArrayList<Country>();
}
```

2. 在使用完之后手动清空ThreadLocal 存储的信息

```
PageHelper.clearPage();
```

3. Java8 lambda用法

```
Page<Country> page = PageHelper.startPage(1, 10).doSelectPage(()-> countryMapper.selectGroupBy());

PageInfo<Country> pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(() -> countryMapper.selectGroupBy());
```


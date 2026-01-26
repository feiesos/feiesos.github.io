+++
date = '2026-01-26T15:48:07+08:00'
draft = false
title = 'Week06'
toc = true
tocBorder = true
+++

# 日志规范
## 使用日志框架API
应用中不可直接只用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架SLFJ中的API，使用门面模式的日志框架，有利于维护和各个类的之日处理方式统一。

## 日志文件保存时间
日志文件推荐至少保存15天，因为有些异常具备以“周”为频次发生的特点。

## 扩展日志命名方式
应用中的扩展日志（如打点、临时监控、访问日志等）命名方式：appName_logType_logName.log。

logType：日志类型，推荐分类有stats/monitor/visit等；logName：日志描述。

这种命名的好处是，通过文件名就可知道日志文件属于什么应用，什么类型，什么目的，也有利于归类查找。

> 正例：mappserver应用中单独监控时区转换异常，如：`mppserver_monitor_timeZoneConvert.log` 

> 说明：推荐对日志进行分类，如将错误日志和业务日志分开存放，便于开发人员查看，也便于通过日志对系统进行及时监控。

## 日志条件输出
对trace/debug/info级别的日志输出，必须使用条件输出形式或者使用占位符的方式。

> 说明：`logger.debug("Processing trade with id: " + id + " and symbol: " + symbol);`如果日志几倍是warn，上述日志不会打印，但是会执行字符串拼接操作，如果sysmbol是对象，会执行`toString()`方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。

> 正例：
> ```java
> (条件)
> if (logger.isDebugEnabled()){
> logger.debug("Processing trade with id: " + id + " and symbol: " + symbol);
> }
> ```
> 或
> ```java
> (占位符)
> logger.debug("Processing trade with id: {} adn sysmol: {} ", id, symbol);
> ```

## 避免重复打印
避免重复打印日志，浪费磁盘空间，务必在log4j.xml中设置additivity=false。

> 正例：`<logger name="com.taobao.dubbo.config" additivity="false">`

## 异常信息日志
异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么通过关键字throws往上抛出。

> 正例：`logger.error(各类参数或者对象toString + "_" + e.getMessage(), e);`

## 谨慎记录日志
谨慎地记录日志。

生产环境禁止输出debug日志；有选择地输出info日志；如果使用warn来记录刚上线时的业务行为信息，一定要注意日志输出量的问题，避免把服务器磁盘撑爆，并记得及时删除这些观察日志。

> 说明：大量地输出无效日志，不利于系统性能提升，也不利于快速定位错误点。记录日志时请思考：这些日志真的有人看吗？看到这条日志你能做什么？能不能给问题排查带来好处？

## 合理使用warn和error级别日志
可以使用warn日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从。如非必要，请不要在此场景打出error级别，避免频繁报警。

> 说明：注意日志输出的级别，error级别只记录系统逻辑出错、异常或者重要的错误信息。
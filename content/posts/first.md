+++
date = '2025-12-11T22:02:02+08:00'
draft = false
title = 'Spring Boot快速入门'
toc = true
tocBorder = true
tags = ["Spring Boot", "Tutorial"]
+++

## 创建项目
打开IDEA，选择**Spring Boot**项目生成器创建一个SpringBoot项目。
1. 填写项目基本信息；
2. 我们选择Java语言、Maven作为依赖管理和项目构建工具，打包方式为Jar；
3. 点击下一步。这里IDEA可为我们自动导入依赖，可以先不选，用到时手动配置即可。

## 项目准备
等待Maven以及其他依赖引入，由于仓库不再国内，我们可以选择配置镜像或者代理的方式加速。

### 配置镜像
1. 打开Maven的配置文件`setting.xml`；
2. 添加
``` xml
<mirrors>
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>
```
3. 这样就配置好了aliyun镜像，接着保存即可。

### 配置代理
或者，如果你有代理，可以通过配置代理的方式加速依赖引入。
1. 打开IDEA设置，选择`Appearance & Behavior` - `System Settings` - `Http Proxy`；
2. 选择`Manual proxy configuration`并根据你的代理选择**HTTP**或者**SOCKS**协议；
3. 填写`Host name`和`Port number`。如果你有需要，填写其他字段；
4. 点击OK保存。

## 启动项目
依赖引入完成，接着我们尝试第一次启动项目。
1. 首先，我们打开`src/main/java/com.example.demo/DemoAplication.java`，当然这里的包名和类名应与你创建时的一致；
2. 接着，使用`Shift` + `F10`运行项目。
3. 运行成功，我们发现Console中输出了
```text
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v4.0.0)

2025-12-11T22:21:18.509+08:00  INFO 28696 --- [demo] [           main] org.example.demo.DemoApplication         : Starting DemoApplication using Java 21.0.8 with PID 28696 (D:\Develop\Web\SpringBoot\demo\target\classes started by f1000 in D:\Develop\Web\SpringBoot\demo)
2025-12-11T22:21:18.511+08:00  INFO 28696 --- [demo] [           main] org.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2025-12-11T22:21:18.570+08:00  INFO 28696 --- [demo] [           main] org.example.demo.DemoApplication         : Started DemoApplication in 0.289 seconds (process running for 0.626)

Process finished with exit code 0
```
这表明运行成功。

下面我们尝试在类中添加一些方法，供我们后续访问：
1. 回到`DemoAplication.java`，这是构建器为我们创建的示例类，里面几乎是空的，只有一个main方法。
2. 我们在类中创建另一个方法hello，它将返回一个字符串：
```Java
public String hello() {
    return "Hello World";
}
```
3. 我们重新启动项目。
通过Console中的日志我们发现项目启动成功。不过我们无法通过浏览器访问hello方法，因为没有Web容器。

## 引入Spring Mvc依赖
为引入Tomcat（web容器），我们手动配置spring mvc依赖。
1. 在`pom.xml`中的`<dependencies></dependencies>`标签中，添加
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
2. 然后同步Maven即可。

## 为hello方法添加请求映射注解
再开始之前，我们先来了解一下**注解**。

### 什么是注解？
注解是一种为程序元素（如类、方法、变量等）添加元数据的机制。它并不会影响程序的运行，而是为编译器、开发工具和运行时提供信息。

简单说，注解就是贴在代码上的“说明标签”，告诉Java或者框架如何处理这段代码。

比如我们将要用到的`@RestController`注解，就是告诉Spring：“这个类是个控制器，方法需要你转成JSON”。

### 为什么要注解？
1. 注解能够减少重复代码，提升代码可读性；
2. 注解作为XML配置方式的一种替代，提供了更简单、直观的配置方法；
3. 注解能够让框架自动处理很多东西。

总之，注解是种强大的工具，合理使用注解可以提升代码可读性、可维护性和自动化程度。

### 注解的分类
在Java中注解分三类，
1. 编译时使用的注解，这将在编译时让编译器检查。\
    比如@Override，这告诉编译器这个方法覆盖了父类方法，否则报错。
2. 运行时使用的注解，这类注解最常见，这些注解在程序运行时依然存在，框架会通过**反射**读取它们，并经过特殊处理。\
    比如@Server，@GetMapping，@AutoWired等。
3. 源码使用的注解，这类注解一般只给开发者看。\
    比如@Deprecated，表示此方法不推荐使用。

了解了注解，我们来配置请求映射。
1. 首先，给DemoApplication类配置@RestController注解。我们上面已经说明了，这个注解能将方法的返回转成JSON。
```Java
@RestController
public class DemoApplication {
    ...
}
```
2. 接着，我们为hello方法添加@RequestMapping注解。这个注解能够回应HTTP请求。
```Java
@RequestMapping("/hello")
public String hello() {
    return "Hello World";
}
```

完成。现在我们启动项目，通过浏览器就能访问hello方法。

服务默认启动在8080端口，在浏览器地址栏输入[`localhost:8080/hello`](http://localhost:8080/hello)来访问hello方法。

如果我们看到“Hello World”，说明配置成功。浏览器通过GET请求访问hello方法，@RestController注解将返回值直接写入HTTP响应体，通常是JSON，但由于方法的返回值是字符串，会直接返回字符串并响应给浏览器。

我们也可以给DemoApplication类加上@RequestMapping注解，比如设为@RequestMapping("/index")，这样做后，请求hello资源的路径不再是/hello，而是/index/hello。简单起见，本例中我们暂不添加。

## 应用配置文件
### 修改服务端口
刚才我们提到了服务默认启动端口，Spring Mvc集成了Tomcat，而Tomcat默认启动在8080。

我们可以通过编辑`src/main/resources/application.properties`来修改，只需添加
```properties
server.port=7891
```
即可。
### 设置根路径
添加`server.servlet.context-path`属性来为当前应用添加根路径。

比如
```properties
server.servlet.context-path=/api
```
这样访问hello路径变为了localhost:7891/api/hello（我们刚刚也改了端口）。

### 补充介绍
application.properties是Spring Boot的核心配置文件，用于配置应用程序的各种参数。Spring Boot也支持application.yml格式。

Spring Boot将按照以下顺序加载配置文件：
1. 项目根目录下的/config子目录
2. 项目根目录
3. classpath下的/config包
4. classpath根目录

application.properties常见配置：
1. 服务器配置，例如端口配置、SSL配置、连接器配置等；
2. 数据库配合，可以数据库url、用户名和密码等；
3. 日志配置。

---
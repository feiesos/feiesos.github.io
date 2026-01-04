+++
date = '2025-12-23T19:20:37+08:00'
draft = false
title = 'Spring Boot笔记'
+++

# SpringBoot3快速入门
## 自动配置机制
### SpringBoot默认扫包规则
`@SpringBootApplication`标注的类就是主程序类。SpringBoot只会扫描主程序所在的包及其子包，相当于自动的component-scan（ssm）。
当然也可以自定义配置扫包：
1. 通过`@SpringBootApplication(scanBasePackages="you.package.path")`注解的scanBasePackages参数配置

    @SpringBootApplication相当于同时使用以下三个注解：
    - SpringBootConfiguration
    - EnableAutoConfiguration
    - ComponentScan

2. 当然也可以通过`ComponentScan("your.package.path")`注解来配置

### 配置文件默认值
- 配置文件的所有配置项是和某个类的对象值进行一一绑定的。这些类我们从称为**配置属性类**。
    
    比如，server.port这个配置项，对应ServerProperties类中的port成员变量。ServerProperties类绑定了所有Tomcat服务器有关的配置。

### 按需加载自动配置
导入场景spring-boot-starter-xxx，场景启动器（starter）除了会导入相关功能以外，还会导入一个`spring-boot-starter`，这是核心场景启动器，是所有starter的starter。

spring-boot-starter导入了一个包spring-boot-autoconfigure。包里面是各种场景的`AutoConfiguration`自动配置类。

虽然全场景的自动配置都在spring-boot-autoconfigure这个包，但是不都是全部开启的，而是按需开启，也就是导入了哪些就开启哪些。

总结：导入场景启动器，触发spring-boot-autoconfigure包的自动配置生效，容器就会具有相关场景。

## 核心技能

### 常用注解
1. 组件注册
    - @Configuration: 告诉Spring容器这是一个配置类（配置类本身也是容器中的组件）
    - @SpringBootConfiguration: 告诉容器这是一个SpringBoot配置类，跟Configuration没区别，仅主要用于主应用类
    - @Bean: 注册一个Bean，组件在容器中的名字默认是方法名，不过可以通过Bean("alias")设置组件名
    - @Controller
    - @Service
    - @Repository
    - @Component
    - @Import
    - @ComponentScan
    - ...

    组件默认是单实例的（每次获取都会获得同一个实例），可以通过@Scoop("prototype")设置组件为多实例（每次获取都会获得一个新的实例）。

    组件注册的步骤：
        1. 配置类注册@Configuration注解
        2. 在配置类中，自定义方法给容器中注册组件（配合@Bean）
        3. 或者使用@Import导入第三方组件

2. 条件注解
如果注解置顶的条件成立，则触发指定行为。

类似@ConditionalOnXxx的注解就是条件注解。

例如@ConditionalOnClass，当类路径中存在这个类，则触发指定行为；相对的@ConditionalOnMissingClass反之。

再比如@ConditionalOnBean，当容器中存在这个Bean（组件），则触发指定行为；相对的还有@ConditionalOnMissingBean。

3. 属性绑定
@ConfigurationProperties和@EnableConfigurationProperties

将容器汇总任意组件（Bean）的属性值和配置文件的配置项的值进行绑定

使用@ConfigurationProperties进项绑定：
    1. 向容器中注册组件（使用@Compoent、@Bean等都可以）
    2. 使用@ConfigurationProperties(prefix="theitemtobind")注解（在需绑定的类或方法上）声明与配置文件中的对应的项（prefix参数的值）的绑定关系

也可以使用@EnableConfigurationProperties进行绑定：
    - 只需在配置类上注解就能`向容器中注册组件`并`绑定属性`，这个注解其实就是整合了@Import和@ConfigurationProperties

由于SpringBoot默认只扫描主程序所在的包，@EnableConfigurationProperties用于第三方组件的绑定，@ConfigurationProperties用于自己写的组件的绑定

## 总结
导入starter --> 生效一系列的xxxAutoConfiguration自动配置类 --> 在容器中注册很多组件 --> 组件参数绑定在xxxProperties属性类中 --> 关联配置文件
                                                                    |                                                         |
                                                            使用Autowired自动装配                                           配置属性

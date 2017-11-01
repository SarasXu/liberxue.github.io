---
layout: blog
technology: true
title:  "Spring声明式事务配置（转）"
background: red
background-image: https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1844720362,2711177956&fm=27&gp=0.jpg
date:   2017-03-09 16:13:54
category: 技术
tags:
- java
- Spring
- 事务
---

Spring Boot 使用事务非常简单，首先使用注解 `@EnableTransactionManagement`开启事务支持后，然后在访问数据库的Service方法上添加注解 `@Transactional` 便可。
关于事务管理器，不管是JPA还是JDBC等都实现自接口`PlatformTransactionManager` 如果你添加的是`spring-boot-starter-jdbc`依赖，框架会默认注入`DataSourceTransactionManager`实例。如果你添加的是`spring-boot-starter-data-jpa`依赖，框架会默认注入 `JpaTransactionManager`实例。
你可以在启动类中添加如下方法，Debug测试，就能知道自动注入的是`PlatformTransactionManager`接口的哪个实现类。
`@EnableTransactionManagement`开启注解事务管理，等同于xml配置方式的` <tx:annotation-driven />`

```java
@SpringBootApplication
public class ProfiledemoApplication {
    @Bean
    public Object testBean(PlatformTransactionManager platformTransactionManager){
        System.out.println(">>>>>>>>>>" + platformTransactionManager.getClass().getName());
        return new Object();
    }
    public static void main(String[] args) {
        SpringApplication.run(ProfiledemoApplication.class, args);
    }
}
```

这些SpringBoot为我们自动做了，这些对我们并不透明，如果你项目做的比较大，添加的持久化依赖比较多，我们还是会选择人为的指定使用哪个事务管理器。 
代码如下：

```java
@EnableTransactionManagement
@SpringBootApplication
public class ProfiledemoApplication {
// 其中 dataSource 框架会自动为我们注入
	@Bean
	public PlatformTransactionManager txManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
    @Bean
    public Object testBean(PlatformTransactionManager platformTransactionManager) {
        System.out.println(">>>>>>>>>>" + platformTransactionManager.getClass().getName());
        return new Object();
    }
    public static void main(String[] args) {
        SpringApplication.run(ProfiledemoApplication.class, args);
    }
}
```

在Spring容器中，我们手工注解`@Bean` 将被优先加载，框架不会重新实例化其他的`PlatformTransactionManager`实现类。
然后在Service中，被 `@Transactional` 注解的方法，将支持事务。如果注解在类上，则整个类的所有方法都默认支持事务。
对于同一个工程中存在多个事务管理器要怎么处理，请看下面的实例，具体说明请看代码中的注释。

```java
@EnableTransactionManagement// 开启注解事务管理，等同于xml配置文件中的 <tx:annotation-driven />
@SpringBootApplication
public class ProfiledemoApplication implements TransactionManagementConfigurer {

    @Resource(name="txManager2")
    private PlatformTransactionManager txManager2;

    // 创建事务管理器1
    @Bean(name = "txManager1")
    public PlatformTransactionManager txManager(DataSource dataSource) {
        returnnew DataSourceTransactionManager(dataSource);
    }

    // 创建事务管理器2
    @Bean(name = "txManager2")
    public PlatformTransactionManager txManager2(EntityManagerFactory factory) {
        returnnew JpaTransactionManager(factory);
    }

    // 实现接口 TransactionManagementConfigurer 方法，其返回值代表在拥有多个事务管理器的情况下默认使用的事务管理器
	@Override
	public PlatformTransactionManager annotationDrivenTransactionManager() {
        return txManager2;
    }
    public static void main(String[] args) {
        SpringApplication.run(ProfiledemoApplication.class, args);
    }
}
@Component
public class DevSendMessage implements SendMessage {
    // 使用value具体指定使用哪个事务管理器@Transactional(value="txManager1")
    @Override
	public void send() {
        System.out.println(">>>>>>>>Dev Send()<<<<<<<<");
        send2();
    }

    // 在存在多个事务管理器的情况下，如果使用value具体指定// 则默认使用方法 annotationDrivenTransactionManager() 返回的事务管理器
	@Transactional 
	public void send2() {
        System.out.println(">>>>>>>>Dev Send2()<<<<<<<<");
    }

}
```

注： 
如果Spring容器中存在多个 `PlatformTransactionManager` 实例，并且没有实现接口 `TransactionManagementConfigurer `指定默认值，在我们在方法上使用注解 `@Transactional` 的时候，就必须要用value指定，如果不指定，则会抛出异常。
对于系统需要提供默认事务管理的情况下，实现接口 `TransactionManagementConfigurer` 指定。
对有的系统，为了避免不必要的问题，在业务中必须要明确指定 `@Transactional` 的 value 值的情况下。不建议实现接口 `TransactionManagementConfigurer`，这样控制台会明确抛出异常，开发人员就不会忘记主动指定。

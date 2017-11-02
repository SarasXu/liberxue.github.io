---
layout: blog
technology: true
istop: true
title:  "SpringBoot项目构建和部署"
background: blue
background-image: https://sarasxu.github.io/winds/images/icon/springboot1.jpeg
date:   2017-03-10 13:41:34
category: 技术
tags:
- java
- SpringBoot
- 部署
- 构建
---

##	SpringBoot简介
Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域（rapid application development）成为领导者。

## 项目构建

使用idea构建一个叫first-boot的maven项目，如图：

![Alt text](https://sarasxu.github.io/blog/img/SpringBoot/1.png)

>因为SpringBoot默认JDK1.8，所以这里选择使用1.8，可以更改JDK版本，但这里不作阐述

![Alt text](https://sarasxu.github.io/blog/img/SpringBoot/2.png)

![Alt text](https://sarasxu.github.io/blog/img/SpringBoot/3.png)

maven项目构建完成

##	添加SpringBoot组件

在建立好的maven项目里的pom文件里面添加SpringBoot的依赖，如下：

```xml
    <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.saras.first-boot</groupId>
    <artifactId>first-boot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--web相关-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.2.0</version>
        </dependency>
        <!-- 导入Mysql数据库链接jar包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>
        <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.28</version>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.saras.firstboot.Main</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

将依赖的组件重新reimport之后，创建一个名为Main的java文件

然后使用SpringBoot的注解`@SpringBootApplication`，并在这个Main文件里写一个main方法，如：

```java
package com.saras.firstboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class);
    }
}
```

写完Main之后，再写一个TestController，如：

```java
package com.saras.firstboot.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class TestController {

    @RequestMapping("")
    @ResponseBody
    public Object hello() {
        return "hello SpringBoot!";
    }
}
```

到这里运行main方法，会报数据源配置的问题，之前加入了mybatis的jar包，需要再配置数据源

创建一个DataSourceConfig的java文件，在里面利用SpringBoot的特性配置bean，如：

```java
package com.saras.firstboot.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import java.sql.SQLException;

@Configuration
public class DataSourceConfig {

    @Primary //默认数据源
    @Bean(name = "dataSource", destroyMethod = "close")
    public DruidDataSource Construction() throws SQLException {
        DruidDataSource dataSource = new DruidDataSource();
        //配置数据连接的参数
        dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/boot?useUnicode=true&characterEncoding=utf8");
        dataSource.setUsername("root");
        dataSource.setPassword("root123");
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");

        //配置最大连接
        dataSource.setMaxActive(20);
        //配置初始连接
        dataSource.setInitialSize(1);
        //配置最小连接
        dataSource.setMinIdle(1);
        //连接等待超时时间
        dataSource.setMaxWait(60000);
        //间隔多久进行检测,关闭空闲连接
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        //一个连接最小生存时间
        dataSource.setMinEvictableIdleTimeMillis(300000);
        //用来检测是否有效的sql
        dataSource.setValidationQuery("select 'x'");
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(false);
        dataSource.setTestOnReturn(false);
        //打开PSCache,并指定每个连接的PSCache大小
        dataSource.setPoolPreparedStatements(true);
        dataSource.setMaxOpenPreparedStatements(20);
        //配置sql监控的filter
        dataSource.setFilters("stat,wall,log4j");
        try {
            dataSource.init();
        } catch (SQLException e) {
            throw new RuntimeException("druid datasource init fail");
        }
        return dataSource;
    }
}

```

这时候运行main方法，访问localhost:8080，如图：

![Alt text](https://sarasxu.github.io/blog/img/SpringBoot/5.png)

一个简单的SpringBoot整合mybatis的项目搭建完成了

## 多环境配置文件

假设有三个环境，local、dev、online。先在resources文件下建立一个叫config的包，在包里创建如下文件：

>application.properties

>application-local.properties

>application-dev.properties

>application-online.properties

将数据库的链接URL、用户名、密码等放到对应环境文件内，再加入一个user.name.hasee的参数，如local：

```properties
user.name.hasee=local
jdbc.connectionURL=jdbc:mysql://127.0.0.1:3306/boot?useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=root123
jdbc.driver=com.mysql.jdbc.Driver
```

新建一个EnvConfig的java文件

```java
package com.saras.firstboot.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;

@Configuration
public class EnvConfig {
    @Value("${user.name.hasee}")
    private String name;
    @Value("${jdbc.connectionURL}")
    private String connectionURL;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;
    @Value("${jdbc.driver}")
    private String driver;

    public String getDriver() {
        return driver;
    }

    public String getConnectionURL() {
        return connectionURL;
    }
    public String getUserName() {
        return userName;
    }

    public String getPassword() {
        return password;
    }

    public String getName() {
        return name;
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}

```

修改DataSourceConfig文件

```java
package com.saras.firstboot.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import java.sql.SQLException;

@Configuration
public class DataSourceConfig {
    @Autowired
    private EnvConfig envConfig;

    @Primary //默认数据源
    @Bean(name = "dataSource", destroyMethod = "close")
    public DruidDataSource Construction() throws SQLException {
        DruidDataSource dataSource = new DruidDataSource();
        //配置数据连接的参数
        dataSource.setUrl(envConfig.getConnectionURL());
        dataSource.setUsername(envConfig.getUserName());
        dataSource.setPassword(envConfig.getPassword());
        dataSource.setDriverClassName(envConfig.getDriver());

        //配置最大连接
        dataSource.setMaxActive(20);
        //配置初始连接
        dataSource.setInitialSize(1);
        //配置最小连接
        dataSource.setMinIdle(1);
        //连接等待超时时间
        dataSource.setMaxWait(60000);
        //间隔多久进行检测,关闭空闲连接
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        //一个连接最小生存时间
        dataSource.setMinEvictableIdleTimeMillis(300000);
        //用来检测是否有效的sql
        dataSource.setValidationQuery("select 'x'");
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(false);
        dataSource.setTestOnReturn(false);
        //打开PSCache,并指定每个连接的PSCache大小
        dataSource.setPoolPreparedStatements(true);
        dataSource.setMaxOpenPreparedStatements(20);
        //配置sql监控的filter
        dataSource.setFilters("stat,wall,log4j");
        try {
            dataSource.init();
        } catch (SQLException e) {
            throw new RuntimeException("druid datasource init fail");
        }
        return dataSource;
    }
}

```

这样使用mvn package打包完成后，即可在服务器中启动项目时指定运行环境对应的配置文件了，如：

>java -Dspring.profiles.active=dev -jar com.saras.firstboot-1.0-SNAPSHOT.jar

不过为了让开发更便捷，还需要做些事

先创建一个名为FirstBootApplication的annotation，如：

```java
package com.saras.firstboot.annotation;

import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootApplication
public @interface FirstBootApplication {
    /**
     * 环境名称
     */
    String env() default "local";

    /**
     * httpPort 默认8080
     */
    int port() default 8080;
}

```

再创建一个名为FirstBoot的java文件，如：

```java
package com.saras.firstboot.base;

import com.saras.firstboot.annotation.FirstBootApplication;
import com.saras.firstboot.utils.Apps;
import org.springframework.boot.SpringApplication;

public class FirstBoot {

    public void run(Class clazz, String... args) {
        long begin = System.currentTimeMillis();
        FirstBootApplication boot = (FirstBootApplication) clazz.getAnnotation(FirstBootApplication.class);
        Apps.setProfileIfNotExists(boot.env());
        Apps.setPort(String.valueOf(boot.port()));
        SpringApplication.run(clazz);
        long end = System.currentTimeMillis();
        System.out.println("********************************************************");
        long time = end - begin;
        System.out.println("启动成功[port：" + boot.port() + "   profile：" + Apps.getProfile() + "   in:" + time + "ms]");
        System.out.println("********************************************************");
    }
}

```

修改Main文件的main方法

```java
package com.saras.firstboot;

import com.saras.firstboot.annotation.FirstBootApplication;
import com.saras.firstboot.base.FirstBoot;

@FirstBootApplication(env = "dev", port = 9094)
public class Main {
    public static void main(String[] args) {
        new FirstBoot().run(Main.class);
    }
}
```

这样就可以直接指定使用的端口和环境

上面使用到的工具类Apps：

```java
package com.saras.firstboot.utils;

import org.springframework.util.StringUtils;

public class Apps {

    private final static String SPRING_PROFILE_ACTIVE = "spring.profiles.active";
    private final static String SERVER_PORT = "server.port";

    public static void setProfileIfNotExists(String profile) {
        if (!StringUtils.hasLength(System.getProperty(SPRING_PROFILE_ACTIVE))
                && !System.getenv().containsKey("SPRING_PROFILES_ACTIVE")) {
            System.setProperty(SPRING_PROFILE_ACTIVE, profile);
        }
    }

    public static void setPort(String port) {
        if (!StringUtils.hasLength(System.getProperty(SERVER_PORT))
                && !System.getenv().containsKey("SERVER_PORT")) {
            System.setProperty(SERVER_PORT, port);
        }
    }

    public static String getProfile() {
        return System.getProperty(SPRING_PROFILE_ACTIVE);
    }
}

```

更改一下TestController看看效果

```java
package com.saras.firstboot.web;

import com.saras.firstboot.config.EnvConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class TestController {
    @Autowired
    private EnvConfig envConfig;

    @RequestMapping("")
    @ResponseBody
    public Object hello() {
        return "hello:" + envConfig.getName();
    }
}

```

运行main方法在控制台可看见

![Alt text](https://sarasxu.github.io/blog/img/SpringBoot/6.png)

访问localhost:9094

![Alt text](https://sarasxu.github.io/blog/img/SpringBoot/7.png)

这样在开发环境配置多环境完成

##	Windows下部署SpringBoot项目

打开CMD进入项目目录

执行 

>mvn clean

成功后执行

>mvn package

在target里面会有XXX.jar和XXX.jar.original

这两个文件一个不能少，如果没有则打包有问题

打包完成后，进入target文件

执行

>java -jar XXX.jar 

项目即部署完成

如果需要指定运行环境，则执行

>java -Dspring.profiles.active=online -jar XXX.jar

这样就算Main文件内指定运行环境为dev，也会被覆盖。

## 结尾

目前还没有在linux上进行部署过，待部署过之后再来写linux的部分，还可以利用脚本简化命令，将打包和启动结合。

[https://github.com/SarasXu/firstboot](https://github.com/SarasXu/firstboot "firstboot")















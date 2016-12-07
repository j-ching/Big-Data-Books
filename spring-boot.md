title: Spring boot入门
notebook: 技术相关
tags: spring, spring-boot

[TOC]

Spring boot方便了用户去创建一个独立的，基于spring的应用，你可以直接运行。大多数的spring boot 应用不需要太多的配置。 

# maven 设置

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.2.RELEASE</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

并不是所有的应用都需要继承直``spring-boot-starter-parent``, 如果你有自己的parent，你可以使用如下的Maven配置

    <dependencyManagement>
         <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.4.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
           </dependency>
        </dependencies>
    </dependencyManagement>

使用``spring-boot-maven-plugin``来打包为可执行的包

# 代码结构
** 建议使用自定义的package，默认的default 包会造成使用``@ComponentScan``, ``@EntityScan``, ``@SpringBootApplication``声明的application 出现问题，每个jar包中的每个class都会被读取**

## application类的位置

应用主Class 建议放在package的根目录下， ``@EnableAutoConfiguration``一般放在主Class之上，同时也隐含着该目录为操作的基本目录， 比如你定义了一个JPA的应用， 则spring会在``@@EnableAutoConfiguration``定义的package中搜索``@Entity``申明的实例。

同时使用一个根目录，可以在申明``@CompanentScan``时不用去设置``basePackage``属性， t同时当主类在根目录的时候， 你也可以使用``@SpringBootApplication`` 注解。

    com
     +- example
         +- myproject
             +- Application.java
             |
             +- domain
             |   +- Customer.java
             |   +- CustomerRepository.java
             |
             +- service
             |   +- CustomerService.java
             |
             +- web
                 +- CustomerController.java

如下``Application.java`` 中申明了主类，同时也设置了``@Configuration``

    package com.example.myproject;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    @EnableAutoConfiguration
    @ComponentScan
    public class Application {

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

    }

# Configuration class

spring boot 喜欢基于java的配置, 尽管``SpringApplication.run()``可以使用xml配置源来启动。建议你的主要配置源是一个``@Configuration`` 的类， 通常将主入口``main()``方法所在的类标记为主``@Configuration``类

## 引入附加的Configuration类
你不需要将所有的``@Configuration``都放在一个单独的类中，``@Import``注解可以引入额外的配置类。 同时你也可以使用``@ComponentScan``来自动的引入Spring的组件，包括``@Configuration``类

## 引入XML 配置
如果你执意要使用xml的配置，那我们建议你使用一个``@Configuration``类，然后通过``@ImportResource``来加载xml的配置

# Auto-Configuration
Spring Boot自动配置功能，可以自动的加载依赖的jar包中的配置。 如``HSQLDB``在你的classpath中， 你不需要手动配置任何的数据库连接，springboot会自动的加载到内存中。

你可以通过``@ENableAutoConfiguration``或者``@SpringBootApplication``注解``@Configuration``类来控制自动加载

## 逐步替换Auto-configuration
自动加载配置功能是非侵害的，如果你需要定义自己的配置来取代自动配置就比较麻烦了。 比如你想添加自己的``DataSource``实例， 默认的配置就需要被置后

在运行应用的时候，使用``--debug``,可以看到当前自动加载的所有配置信息。

## 让指定配置的自动加载失效
可以使用``@EnableAutoConfiguration``的exclude属性来指定某些配置自动加载失效

    @Configuration
    @EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
    public class MyConfiguration {
    }

如果类不在你的classpath中，可以使用``excludeName``属性指定全名。 同样你可以使用``spring.autoconfigure.exclude``来指定一个列表

# Spring Bean 以及依赖注入
你可以很容易的使用任何Sprping Framework的技术来定义你的bean，并注入。 通过``@ComponentScan`` 查找你的beans，然后通过``@Autowired``来注入

如果你按照之前我们建议的那样，将应用的主类放在根目录下， 那你可以使用没有任何参数的``@ComponentScan``， 应用中所有的组件(``@Componenet``, ``@Service``,``@Repository``,``@Controller``等)都会被自动注册
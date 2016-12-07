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



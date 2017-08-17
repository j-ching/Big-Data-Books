[TOC]
# Spring4.3.7特征

## 注解 
+ **@RestController** 与``@Controller``类似，告知sprint这是个类是用来处理web请求的，不同的是，``@RestController``告诉spring将结果直接渲染到调用端并显示，功能相当于``@Controller``和``@ResponseBody``结合
+ **@EnableAutoConfiguration** 

+ **SpringBootApplication** 等于 ``@Configuration``,``@EnableAutoConfiguration`` 和``@ComponentScan``三个注解的结合


## 注意事项
+  在webapp中，通常我们会把静态文件放在/webapp目录下， 但是这个目录只是在我们war包的时候有效。 使用spring-boot，将运行程序打为jar包的时候，常见的是将静态文件放在/src/main/resources/static

## spring 开发创建非web应用

1. 需要将``springBootApplication``设置为非web环境

        springBootApplication.setWebEnvironment(false)
    
    或者设置``applicationContextClass``属性
    
2. 你执行的逻辑层实现``CommandLineRunner``接口，如果有多个逻辑层的类，你可以通过``order``注解来指定执行的顺序
3. ``ApplicationRunner``和``CommandLineRunner``在``SpringApplication``启动之前执行


  ##  @Value中 @与#的区别


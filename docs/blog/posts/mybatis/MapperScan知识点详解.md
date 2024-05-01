---
date: 2024-01-31
authors:
  - chenwp
categories:
  - MyBatis
---



# MapperScan知识点详解

## 一、简介

@MapperScan是MyBatis框架中的一个注解，它的作用是**用于扫描MyBatis Mapper接口，并将这些接口注册到MyBatis框架中** 它通过MapperScannerRegistrar执行与MapperScannerConfigurer相同的工作。

可以指定basePackageClasses 或basePackages（或其别名值）来定义要扫描的特定包。 从2.0.4开始，如果没有定义特定的包，将从声明该注解的类的包开始扫描。



<!-- more -->

## 二、使用

将Mybatis Mapper接口注册到spring容器中有两种常用方式：@Mapper和@MapperScan。

- @Mapper

添加在Mapper接口类中，将该接口注册到容器

- @MapperScan

 一般添加在springboot启动类中，通过配置Mapper接口的路径，将该路径下的接口都注册到容器中。

> 注：如果同时使用两个注解，会导致@Mapper失效。

### 2.1 使用示例

```
@SpringBootApplication
@MapperScan("com.boot3.dynamicDataSource.mapper")
public class ApplicationStater {

    public static void main(String[] args) {
        SpringApplication.run(ApplicationStater.class, args);
    }

}
```

## 三、源码分析

### 3.1 MapperScan注解

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(MapperScannerRegistrar.class)
@Repeatable(MapperScans.class)
public @interface MapperScan {

  @AliasFor("basePackages")
  String[] value() default {};

  @AliasFor("value")
  String[] basePackages() default {};

  Class<?>[] basePackageClasses() default {};

  ....省略其他属性
}
```

MapperScan注解中重要的就是@Import注解，它用来将类注册Spirng的IOC容器中。

### 3.2 Import注解

Import注解提供三种方式可以将类注册到IOC容器：

```
@Import一个普通类 spring会将该类加载到spring容器中

@Import一个类，该类实现了ImportBeanDefinitionRegistrar接口，在重写的registerBeanDefinitions方法里面，能手工往beanDefinitionMap中注册beanDefinition

@Import一个类 该类实现了ImportSelector 重写selectImports方法该方法返回了String[]数组，所有保存到数组中的全路径类名都将被注册到IOC容器中
```

### 3.3 MapperScannerRegistrar
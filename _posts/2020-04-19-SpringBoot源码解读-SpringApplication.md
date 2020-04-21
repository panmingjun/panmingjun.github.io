---
title: SpringBoot源码解读-SpringApplication
layout: post
category: SpringBoot
---

>SpringBoot可以说是目前为止应用最广泛的java框架之一了,不但功能强大,而且使用简单.  
对于新人来说,可以不用管它具体是如何实现的.但是对于一个有理想抱负的程序员,理解其内部的原理是非常的有必要.  
这篇文章就是我个人对于SpringBoot的研究理解,并不是权威,如有错误,还请指正.  
使用的jar包版本为2.2.0.RELEASE


## 1.SpringApplication

用过SpringBoot的人都认得下面这段代码:

``` java
@SpringBootApplication
public class Application {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class,args);
    }
}
```
归根结底,我们知道```SpringApplication.run(Application.class,args);```和```new SpringApplication(Application.class).run(args);```是一样的

首先来看构造函数内部发生了什么:

### 1.1 构造函数

构造函数非常简单,可以看到主要就是给```SpringApplication```内的各种字段赋值

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;//Spring的资源加载接口,主要功能就是提供统一的接口来访问磁盘资源和网络资源等各种io流,还有提供类加载器.
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));//SpringBoot的主类,注意这里不是main函数所在的类,而是@SpringBootApplication注解的类
		this.webApplicationType = WebApplicationType.deduceFromClasspath();//3是启动的应用类型
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));//
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();//和primarySource不同,这个才是main方法所在的类
	}
```

下面就来简单探究一下字段的用处.

### 1.1.1 resourceLoader

```ResourceLoader```是一个接口,内部申明两个接口,一个是获取资源的getResource,一个是获取类加载器.可以说大多数情况下我们都不需要修改这两个方法,当你没有给该字段赋值的时候,SpringBoot默认会使用```org.springframework.core.io.DefaultResourceLoader```这个类来加载资源.

我们知道SpringBoot的启动banner是可以通过banner.txt这个文件来指定内容的,而这个banner.txt就是通过resourceLoader字段的getResource方法读取的,这里我们可以来做个试验:
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) throws Exception {
        new SpringApplicationBuilder()
                .sources(Application.class)
                .resourceLoader(new DefaultResourceLoader() {
                    @Override
                    public Resource getResource(String location) {
                        Resource resource = super.getResource(location);
                        if (!resource.exists()) {
                            resource = new FileSystemResource("/Users/panmingjun/Desktop/" + location);
                        }
                        return resource;
                    }

                    @Override
                    public ClassLoader getClassLoader() {
                        return super.getClassLoader();
                    }
                })
                .run(args);
    }
}
```
我们可以重写一个ResourceLoader继承```DefaultResourceLoader```,当```DefaultResourceLoader```读取不到文件时,会自动从指定的"Desktop"目录读取文件.  
然后我们在"Desktop"目录下创建一个banner.txt,运行后就会读取banner.txt的内容并打印到控制台了.

未完成...
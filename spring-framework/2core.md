<!-- TOC -->

- [概览](#%E6%A6%82%E8%A7%88)
  - [1 IOC容器](#1-ioc%E5%AE%B9%E5%99%A8)
    - [1.1 介绍Spring的IOC容器和Beans](#11-%E4%BB%8B%E7%BB%8Dspring%E7%9A%84ioc%E5%AE%B9%E5%99%A8%E5%92%8Cbeans)
    - [1.2 容器概览](#12-%E5%AE%B9%E5%99%A8%E6%A6%82%E8%A7%88)
      - [1.2.1 配置元数据](#121-%E9%85%8D%E7%BD%AE%E5%85%83%E6%95%B0%E6%8D%AE)
      - [1.2.2 实例化一个Spring 容器](#122-%E5%AE%9E%E4%BE%8B%E5%8C%96%E4%B8%80%E4%B8%AAspring-%E5%AE%B9%E5%99%A8)
      - [1.2.3 使用Spring 容器](#123-%E4%BD%BF%E7%94%A8spring-%E5%AE%B9%E5%99%A8)
    - [1.3 Bean概览](#13-bean%E6%A6%82%E8%A7%88)

<!-- /TOC -->

#  概览

本文讲解Spring的核心技术，版本为5.1.7.RELEASE

本文包含Spring的所有核心技术

Spring中最重要的就是IOC容器。在讲解了IOC之后紧接着我们会讲解AOP技术。Spring也有自己的AOP技术，Spring的AOP概念简单易懂，并且  
涵盖了80%的企业需求。

Spring还提供了整合AspectJ的功能。AspectJ是当前企业中最成熟的AOP方案

## 1 IOC容器

本章讲解Spring的IOC容器。

### 1.1 介绍Spring的IOC容器和Beans

```
本章讨论Spring IOC的设计理念。IOC也称之为DI。这是一个通过构造方法、对象定义等方法定义依赖关系，然后容器在创建bean时通过这些依赖关系进行处理。这和业务代码自己实现管理依赖关系刚好相反，所以称之为依赖反转。

org.springframework.beans和org.springframework.context是IOC依赖的最基础的包。BeanFactory接口提供了一种能够管理任何类型对象的高级配置机制。ApplicationContext是BeanFactory的子接口。它提供以下功能:
1. 集成AOP更加简单便捷
2. 消息资源处理(比如针对国际化)
3. 事件的发布和订阅
4. 应用级别指定的上下文，比如WebApplicationContext
简单说: BeaFactory提供了最基本的配置框架和基本功能，ApplicationContext提供更多企业级的特性。ApplicationContext是
BeanFactory的完整超集，在本章中仅用于Spring的IoC容器的描述。如果要查询更多功能，请参见 BeanFactory章节

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。Bean的实例化、组装等生命周期的管理工作都是Spring IOC容器
提供能力
```

### 1.2 容器概览

```
org.springframework.context.ApplicationContext代表了Spring的IOC容器，主要负责bean的初始化、配置及组装。
容器从自己的配置元数据中获取到哪些对象需要初始化、配置以及组装。配置的元数据包括XML、Java注解及Java代码。他依赖于你提供的应用程序
对象以及其依赖

Spring提供了ApplicationContext接口的几个实现，在独立部署的应用中，通常会创建ClassPathXmlApplicationContext或
FileSystemXmlApplicationContext。虽然xml是种传统方案提供配置元数据，你也可以使用Java注解和代码为Spring提供元数据，这样可以简化元素据的配置量
在大多数应用程序方案中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。例如一个web应用的脚本，应用程序的web.xml文件中的简单八行（左右）样板Web描述符XML通常就足够了。在使用IDE的情况下更简单
下图显示了Spring如何工作的高级视图。您的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，您拥有一个完全
配置且可执行的系统或应用程序。
```
![container](img/1-container-magic.png)

#### 1.2.1 配置元数据

```
在上一章的图表中可以发现，Spring IOC容器会消费一个格式化的配置文件。这个配置文件代表了开发者制定的初始化、配置及组装的规则。
传统的配置文件使用xml形式，本文也将采用xml的形式对Spring IOC容器进行讲解

备注: 基于XML的配置文件不是Spring支持的唯一格式，当前更多的开发者选择使用Java注解进行配置。基于注解和基于代码的方式讲解如下:
1. 基于注解: 从Spring 2.5开始支持基于注解的配置方式
2. 基于编码: 从Spring 3.0开始支持基于代码的配置方式，可以使用@Configuration, @Bean, @Import, and @DependsOn 等注解

Spring配置包含容器必须管理的至少一个且通常不止一个bean定义。对应到XML配置的话就是<bean>注解。代码的实现可以参照@Bean和
@Configuation注解

这些bean定义对应于构成应用程序的实际对象。通常，您定义服务层对象，数据访问对象（DAO）。很多时候代表Struts Action实例，基础结构对象，代表Hibernate SessionFactories，JMS队列等。通常，不会在容器中配置细粒度域对象，因为DAO和业务逻辑通常负责创建和加载域对象。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。
下例就是基于xml的配置对象

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">   
        <!-- collaborators and configuration for this bean go here -->
    </bean>
    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions go here -->
</beans>
```

#### 1.2.2 实例化一个Spring 容器

```
提供给ApplicationContext构造函数的位置路径是资源字符串，它允许容器从各种外部资源（如本地文件系统，Java CLASSPATH
等）加载配置元数据。
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

```

#### 1.2.3 使用Spring 容器

### 1.3 Bean概览


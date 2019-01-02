# spring-core
### Spring Framework Documentation

>  `Core` : ioc 容器，事件，资源，i18n，验证，数据绑定，类型转换，spel, aop。

## **1.1 Spring IoC容器和Bean简介**

Ioc也成为依赖注入（DI）。这是一个过程，通过这个过程，对象只能通过

构造函数参数

工厂方法构造

或返回的对象实例上设置的属性来定义它们的依赖关系。

然后容器在创建bean时注入这些依赖项。

这个过程基本上是bean本身通过使用类的直接构造或服务定位器模式之类的机制来控制其依赖关系

的实例化或位置的逆，即因此名次，为控制反转。

-----

**1.1.1 理解**

在`org.springframework.beans` 和 `org.springframework.context` 包是Spring 框架的IoC容器的基础。

该BeanFactory 接口提供一种能够管理任何类型对象的高级配置机制。 ApplicationContext 是一个子接口BeanFactory.扩展：

1. 更容易与Spring的AOP功能集成
2. 消息资源处理（用于国际化）
3. 事件处理
4. `WebApplicationContext` 于 应用程序层的上下文，例如在Web应用程序中使用的上下文。

简而言之，BeanFactory 提供了配置框架和基础功能，并`ApplicationContext`添加了更多企业特定的功能。

**1.1.2 Bean 简介**

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是一个由Spring IoC容器实例化，组装和管理的对象。否则，bean只是应用程序中众多对象之一。Bean及其之间的依赖关系反映在容器使用的配置元数据中。

## 1.2 集装箱概览

`org.springframework.context.Application` 接口表示Spring IoC容器，负责实例化，配置和组装bean。

`ApplicationContext` 

Spring提供了几种接口实现：

`ClassPathXmlApplicationContext` 

`FileSystemXmlApplicationContext`

 Spring IoC容器图

![1546413829425](C:\Users\sunyang\AppData\Local\Temp\1546413829425.png)

上图显示了Spring如何工作。

**1.2.1 配置元数据**

1. 基于注释的配置（Spring 2.5 引入了对基于注释的配置元数据的支持）
2. 基于java的配置（Spring 3.0 开始）@Configuration, @Bean, @Import 等

**1.2.2 实例化容器**

~~~java
/**
* Application 构造函数的位置路径是资源字符串，
* 它允许容器从各种外部资源加载配置元数据 ClassPath
*/
ApplicationContext context =  new ClassPathXmlApplication("services.xml", "daos.xml");
~~~

服务层对象（`services.xml`）配置文件：

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->
</beans>
~~~

数据访问对象`daos.xml` 文件：

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->
</beans>
~~~

**编写基于XML的配置元数据**

bean定义跨越多个XML文件。`<import/>`

~~~xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
~~~

**1.2.3 使用容器**

`Application`是高级工厂的接口，能够维护不同bean及其依赖项的注册表。

通过`T getBean(String name, Class<T> requiredType)`，可以检索Bean的实例。

~~~java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
PetStoreService service = context.getBean("petStore", PetStoreService.class);
List<String> userList = service.getUserNameList();
~~~



## 1.3 Bean概述

1.在spring IoC容器管理一个或多个bean，这些bean是使用你提供给容器的配置元数据创建的。

2.在容器本身内，这些bean定义标识为BeanDefinition对象，其中包含以下元数据：

- 包限定的类名：通常是正在定义的bean的实际实现类。
- Bean行为配置元素，说明bean在容器中的行为方式（范围，生命周期回调等）。
- 引用bean执行其工作所需的其他bean。这些引用也称为协作者或依赖项。
- 要在新创建的对象中设置的其他配置设置，例如池的大小限制或在管理连接池的Bean中使用的连接数。

bean定义：

| 属性           |
| -------------- |
| 类             |
| 名称           |
| 范围           |
| 构造函数参数   |
| 属性           |
| 自动装配模式   |
| 延迟初始化模式 |
| 初始化方法     |
| 销毁方法       |

1.3.1 bean命名

~~~html
Bean命名约定
惯例是在命名bean时使用标准Java约定作为实例字段名称。也就是说，bean名称以小写字母开头，并从那里开始驼峰。这样的名字的例子包括accountManager， accountService，userDao，loginController，等等。

命名bean始终使您的配置更易于阅读和理解。此外，如果您使用Spring AOP，那么在将建议应用于与名称相关的一组bean时，它会有很大帮助。
~~~

**Bean定义之外别名**

~~~java
<alias name="fromName" alias="toName" />
~~~

引用同一对象

~~~java
<alias name="myApp-dataSource" alias="subA-dataSource" />
<alias name="myApp-dataSource" alias="subB-dataSource" />
<alias name="myApp-dataSource" alias="subC-dataSource" />
~~~


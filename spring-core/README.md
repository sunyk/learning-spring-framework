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

**1.3.2 实例化bean类**

* 使用构造函数实例化

当通过构造方法创建bean时，所有普通类都可以使用，并与spring兼容。简单的指定bean类就足够了。但是

> 题外话，为特定bean使用的IoC类型，可能需要一个默认空构造函数。

Spring IoC容器几乎可以管理你希望它管理的任何类。

使用基于XML的配置元数据，你可以按如下方式指定bean类：

~~~xml
<bean id="exampleBean" class="examples.ExampleBean" />
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
~~~



* 使用静态工厂方法实例化

bean定义指定通过调用工厂方法来创建bean。

~~~xml
<bean id="clientService" 
      class="examples.ClientService" 
      factory-method="createInstance"/>//静态工厂方法
~~~

显示可以使用前面的bean定义的类：

~~~java
public class ClientService{
    private static ClientService clientService = new clientService();
    private ClientService(){}
    
    public static ClientService createInstance(){
        return clientService;
    }
}
~~~



* 使用实例工厂方法实例化

~~~xml
<!-- 工厂 bean, 其中包含一种称为 createInstance () 的方法 -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    ...
</bean>

<bean id="clientService"
      factory-bean="serviceLocator"
      factory-method="createClientServiceInstance" />
~~~

显示相应的Java类：

~~~java
public class DefaultServiceLocator{
    
    private static ClientService clientService = new ClientServiceImpl();
    
    public ClientService createClientServiceInstance(){
        return clientService;
    }
}
~~~

> 一个工厂类可以包含多个工厂方法

~~~xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    ...
</bean>

<bean id="clientService"
      factory-bean="serviceLocator"
      factory-method="createClientServiceInstance"/>

<bean id="accountService"
      factory-bean="serviceLocator"
      factory-method="createAccountServiceInstance"/>

~~~

相应的Java类：

~~~java
public class DefaultServiceLocator{
    
    private static ClientService clientService = new ClientServiceImpl();
    
    private static AccountService accountService = new AccountServiceImpl();
    
    public ClientService createClientServiceInstance(){
        return clientService;
    }
    
    public AccountService createAccountServiceInstance(){
        return accountService;
    }
}
~~~

## 1.4 依赖

概念：典型的企业级应用程序不包含单个对象或Spring用法中的bean。即使是最简单的应用程序也有一些对象可以协同工作，以呈现最终用户所看到的连贯的应用程序。

**1.4.1 依赖注入**

依赖注入DI 是一个过程，通过这个过程，对象只能通过

构造函数参数，

工厂方法的参数，

或在构造对象实例后再对象实例上设置的属性来定义它们的依赖关系。

从工厂方法返回，然后容器在创建bean时注入这些依赖项。

这个过程基本上是bean本身的反向（因此名称，控制反转），

通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。

> 好处：使用DI原则的代码更清晰，当对象提供其依赖项时，解耦更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

![1546501770013](C:\Users\sunyang\AppData\Local\Temp\1546501770013.png)



**DI存在两个主要变体：**

- 基于构造函数的依赖注入
- 基于Setter的依赖注入

**基于构造函数的依赖注入**

示例:一个只能使用构造函数注入进行依赖注入的类：

~~~java
public class SimpleMovieLister{
    
    private MovieFinder movieFinder;
    
    public SimpleMovieLister(MovieFinder movieFinder){
        this.movieFinder = movieFinder;
    }
    ...
}
~~~

有参构造器

~~~java
package x.y;

public class ThingOne{
    
    public ThingOne(ThingTwo thingTwo, ThingThree thingThree){
        ...
    }
}

~~~

> 分析：假设`ThingTwo`并且`ThingThree`类与继承无关，则不存在潜在的歧义。因此，以下配置工作正常，您不需在<constructor-arg/>元素中显式指定构造函数参数索引或类型。

~~~xml
<beans>
    <bean id="thingOne" class="x.y.ThingOne">
        <constructor-arg ref="thingTwo"/>
        <constructor-arg ref="thingThree"/>
    </bean>
    
    <bean id="thingTwo" class="x.y.ThingTwo"/>
    <bean id="thingThree" class="x.y.ThingThree"/>
</beans>
~~~

**构造函数参数类型匹配**

~~~java
package examples;
 
public class ExampleBean{
    
    private int years;
    
    private String ultimateAnswer;
    
    public ExampleBean(int years,String ultimateAnswer){
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
~~~

使用`type`属性显式指定构造函数参数的类型，则容器可以使用与简单类型匹配的类型。

~~~xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="22" />
    <constructor-arg type="java.lang.String" value="2019" />
</bean>
~~~

使用`index`属性显式指定构造函数参数的索引。(指定索引还可以解决构造函数具有相同类型的两个参数的歧义)

~~~xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="22" />
    <constructor-arg index="1" value="2019" />
</bean>
~~~

使用构造函数参数名称。(通过参数名称进行消除歧义)

~~~xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="22" />
    <constructor-arg name="ultimateAnswer" value="2019" />
</bean>
~~~

**最终：**为了使这项工作开箱即用，必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。还可以使用`@ConstructorProperties` JDK批注显式命名构造函数参数。

~~~java
package examples;

public class ExampleBean{
    
    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer){
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
~~~

**基于Setter的依赖注入**

示例：一个只能通过使用setter注入进行依赖注入的类。

~~~java
public class SimpleMovieLister{
    
    private MovieFinder movieFinder;
    
    public void setMovieFinder(MovieFinder movieFinder){
        this.movieFinder = movieFinder;
    }
    
    ...
        
}
~~~

### 循环依赖

**场景分析**

主要使用构造函数注入，则可以创建无法解析的循环依赖关系场景。

例如：类A通过构造函数注入需要类B的实例，而类B通过构造函数注入需要类A的实例。

如果将A类和B类的bean配置为相互注入，则Spring IoC容器会再运行时检测此循环引用，并抛出一个`BeanCurrentlyInCreationException`。

**解决方案**

一种可能的解决方案是编辑由setter而不是构造函数配置的某些类的源代码。

或者，避免构造函数注入并仅适用setter注入。

与经典情况（鸡与鸡蛋场景）不同，bean A和 bean B之间的循环依赖强制其中一个bean在完全初始化之前被注入另外一个bean。

-------------------------

**依赖注入的示例**

基于XML的配置元数据用于基于setter的DI。

~~~xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- 使用嵌套 ref 元素的 setter 注入 -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>
    <!-- 使用更近的 ref 属性的 setter 注入 -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1" />
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean" />
<bean id="yetAnotherBean" class="examples.YetAnotherBean" />
~~~

对应的`ExampleBean` 类

~~~java
public class ExampleBean{
    
    private AnotherBean beanOne;
    
    private YetAnotherBean beanTwo;
    
    private int i;
    
    public void setBeanOne(AnotherBean beanOne){
        this.beanOne = beanOne;
    }
    
    public vid setBeanTwo(YetAnotherBean beanTwo){
        this.beanTwo = beanTwo;
    }
    
    public void setIntegerProperty(int i){
        this.i = i;
    }
    
}
~~~

setter被声明为与XML文件中指定的属性匹配。（下面示例是使用基于构造函数的DI）

~~~xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>
    
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg type="int" value="1"/>
    
</bean>
    <bean id="anotherExampleBean" class="examples.AnotherBean"/>
    <bean id="yetAnotherBean" class="examples.YetAnotherBean" />
~~~

对应的`ExampleBean` 类：

~~~java
package  examples.ExampleBean;
    
public class ExampleBean {
    
    ...
   
    public ExampleBean(AnotherBean anotherBean,YetAnotherBean yetAnotherBean, int i){
        this.anotherBean = anotherBean;
        this.yetAnotherBean = yetAnotherBean;
        this.result = i;
    }
    
}
~~~

示例，调用static静态工厂方法来返回对象的实例：.

~~~xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherBean"/>
    <constructor-arg ref="yetAnotherBean" />
    <constructor-arg value="1"/>
</bean>
<bean id="anotherBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean" />
~~~

相应的`ExampleBean` 类：

~~~java
package examples.ExampleBean;

public class ExampleBean{
    
    private ExampleBean((AnotherBean anotherBean, YetAnotherBean yetAnotherBean,int i){
        this.anotherBean = anotherBean;
            this.yetAnotherBean = yetAnotherBean;
            this.result = i;
    }
    
    public static ExampleBean createInstance(){
        public CreateInstance(AnotherBean anotherBean, YetAnotherBean yetAnotherBean,int i){
            ExampleBean eb = new ExampleBean(anotherBean, yetAnotherBean, i);
            //可以做一些操作 。。。
            return eb;
        }
    }
}
~~~














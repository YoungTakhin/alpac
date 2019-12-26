---
title: Spring框架入门2. Spring IoC容器
color: '#EBF8FF'
abbrlink: 865c5d02
date: 2019-12-20 11:41:02
tags:
  - Java
  - Spring
categories: Spring框架入门
---

# Spring框架入门2. Spring IoC容器

## 控制反转

**控制反转**（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI）。

通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递（注入）给它。

## IoC容器

Spring 容器是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。Spring 容器使用依赖注入（DI）来管理组成一个应用程序的组件。这些对象被称为 Spring Beans。

通过阅读配置元数据提供的指令，容器知道对哪些对象进行实例化，配置和组装。配置元数据可以通过 XML，Java 注解或 Java 代码来表示。下图是 Spring 如何工作的高级视图。 Spring IoC 容器利用 Java 的 POJO 类和配置元数据来生成完全配置和可执行的系统或应用程序。

![IoC容器](https://s2.ax1x.com/2019/12/20/QOnMss.png)

**IoC 容器**是具有依赖注入功能的容器，Spring 提供了以下两种不同类型的容器。

|容器|描述|
| --- | --- |
| BeanFactory | 该容器提供了框架和基本功能的配置 |
| ApplicationContext | 该容器添加了更多的企业特定的功能 |

### BeanFactory

`BeanFactory` 是一个最简单的容器，它主要的功能是为依赖注入（DI）提供支持，这个容器接口在 `org.springframework.beans.factory.BeanFactor` 中被定义。`BeanFactory` 和相关的接口，比如 `BeanFactoryAware`、`DisposableBean`、`InitializingBean`，仍旧保留在 Spring 中，主要目的是向后兼容已经存在的和那些 Spring 整合在一起的第三方框架。

在 Spring 中，有大量对 `BeanFactory` 接口的实现。其中，最常被使用的是 `XmlBeanFactory` 类。这个容器从一个 XML 文件中读取配置元数据，由这些元数据来生成一个被配置化的系统或者应用。

在资源宝贵的移动设备或者基于 applet 的应用程序中，优先选择 `BeanFactory`。否则，一般使用的是 `ApplicationContext`。

### ApplicationContext

`ApplicationContext` 是 `BeanFactory` 的子接口，也被成为 Spring 上下文。

`ApplicationContext` 是 Spring 中较高级的容器。和 `BeanFactory` 类似，它可以加载配置文件中定义的 bean，将所有的 bean 集中在一起，当有请求的时候分配 bean。另外，它增加了企业特定的功能，比如，从属性文件中解析文本信息和将事件传递给所指定的监听器。这个容器接口在 `org.springframework.context.ApplicationContext` 接口中被定义。

`ApplicationContext` 包含 `BeanFactory` 所有的功能，一般情况下，相对于 `BeanFactory`，`ApplicationContext` 会更加优秀。当然，`BeanFactory` 仍可以在轻量级应用中使用，比如移动设备或者基于 applet 的应用程序。

`ApplicationContext` 接口最常被使用的实现类：

- `ClassPathXmlApplicationContext`：该容器从类路径下的配置文件中加载已被定义的 bean。使用该容器不需要提供配置文件的完整路径，只需正确配置 CLASSPATH 环境变量即可。
- `FileSystemXmlApplicationContext`：该容器从磁盘任意路径下的配置文件中加载已被定义的 bean。使用该容器需要注意路径必须要有访问权限。
- `WebXmlApplicationContext`：该容器会在一个 Web 应用程序的范围内加载在配置文件中已被定义的 bean。
- `AnnotationConfigApplicationContext`：它是用于读取注解创建容器的。

### BeanFactory 和 ApplicationContext 的区别

- `BeanFactory` 才是 Spring 容器中的顶层接口。`ApplicationContext` 是它的子接口。

- 创建对象的时间点不一样：
  - `ApplicationContext`：它在构建核心容器时，创建对象采取的策略是采用立即加载的方式。也就是说，只要一读取完配置文件马上就创建配置文件中配置的对象。
  - `BeanFactory`：它在构建核心容器时，创建对象采取的策略是采用延迟加载的方式。也就是说，什么时候根据 id 获取对象了，什么时候才真正的创建对象。

## 基于 XML 的 IoC 环境搭建

1. Maven pom.xml 配置
  {% codeblock pom.xml lang:XML %}
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>org.ydx</groupId>
        <artifactId>spring-learn</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>jar</packaging>
    
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>5.2.2.RELEASE</version>
            </dependency>
        </dependencies>
    
    </project>
  {% endcodeblock %}

2. 编写程序代码
  - 编写 dao 层接口及其实现类
    {% codeblock AccountDao.java lang:Java %}
        package com.ydx.dao;
        
        /**
         * 账户的持久层接口
         */
        public interface AccountDao {
        
            /**
             * 模拟保存账户
             */
            void saveAccount();
        }
    {% endcodeblock %}
    
    {% codeblock AccountDaoImpl.java lang:Java %}
        package com.ydx.dao.impl;
        
        import com.ydx.dao.AccountDao;
        
        /**
         * 账户的持久层实现类
         */
        public class AccountDaoImpl implements AccountDao {
        
            public AccountDaoImpl() {
                System.out.println("AccountDaoImpl对象创建了");
            }
        
            public void saveAccount() {
                System.out.println("保存了账户");
            }
        }
    {% endcodeblock %}
    
  - 编写 service 层接口及其实现类
    {% codeblock AccountService.java lang:Java %}
        package com.ydx.service;
        
        /**
         * 账户业务层的接口
         */
        public interface AccountService {
        
            /**
             * 模拟保存账户
             */
            void saveAccount();
        }
    {% endcodeblock %}
    
    {% codeblock AccountServiceImpl.java lang:Java %}
        package com.ydx.service.impl;
        
        import com.ydx.dao.AccountDao;
        import com.ydx.service.AccountService;
        
        /**
         * 账户的业务层实现类
         */
        public class AccountServiceImpl implements AccountService {
        
            private AccountDao accountDao ;
        
            public AccountServiceImpl() {
                System.out.println("AccountServiceImpl对象创建了");
            }
        
            public void  saveAccount() {
                accountDao.saveAccount();
            }
        }
    {% endcodeblock %}

3. resourse 目录下新建 applicationContext.xml
  {% codeblock applicationContext.xml lang:XML %}
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
                    https://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <!--把对象的创建交给Spring来管理-->
    
        <bean id="accountService" class="com.ydx.service.impl.AccountServiceImpl"/>
    
        <bean id="accountDao" class="com.ydx.dao.impl.AccountDaoImpl"/>
    
    </beans>
  {% endcodeblock %}
    
  > id 属性是标识单个 bean 定义的字符串。
  > class 属性使用全限定类名定义 bean 的类型。

4. 编写测试类，调用 service 接口
  {% codeblock BaseTest.java lang:Java %}
      package com.ydx;
      
      import com.ydx.dao.AccountDao;
      import com.ydx.service.AccountService;
      import org.springframework.context.ApplicationContext;
      import org.springframework.context.support.ClassPathXmlApplicationContext;
      
      public class BaseTest {
      
          /**
           * 获取Spring的IoC核心容器，并根据id获取对象
           * @param args
           */
          public static void main(String[] args) {
              //1 获取核心容器对象
              ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
      
              //2 根据id获取bean对象
              AccountService as = (AccountService) ac.getBean("accountService");
              AccountDao ad = ac.getBean("accountDao", AccountDao.class);
      
              System.out.println(as);
              System.out.println(ad);
          }
      }
  {% endcodeblock %}
  
  > 这里使用了 `getBean()` 方法的两种用法，`getBean(String name)` 和 `getBean(String name, Class requiredType)`
  
  运行结果：
  
  ```console
    AccountServiceImpl对象创建了
    AccountDaoImpl对象创建了
    com.ydx.service.impl.AccountServiceImpl@57baeedf
    com.ydx.dao.impl.AccountDaoImpl@343f4d3d
  ```

---
title: Spring框架入门3. Spring Bean
color: '#EBF8FF'
tags:
  - Java
  - Spring
categories: Spring框架入门
abbrlink: 9cc8ab67
date: 2019-12-20 18:07:27
---

# Spring框架入门3. Spring Bean

## Bean

bean 是被实例化，组装，并通过 Spring IoC 容器所管理的构成应用程序支柱的对象。这些 bean 是由用容器提供的配置元数据创建的

在容器内部，这些 **bean 定义**被表示为 ```BeanDefinition``` 对象，其中包含以下元数据(以及其他信息):

  - 全限定类名：通常是定义的 bean 的实际实现类。
  - bean 行为配置元素：它表示 bean 在容器中的行为(范围、生命周期回调，等等)。
  - 依赖：对该 bean 执行其工作所需的其他 bean 的引用。
  - 新创建的对象中的其他配置设置：例如，池的大小限制或管理连接池的 bean 中使用的连接数。

上述所有的配置元数据转换成一组构成每个 **bean 定义**的下列属性：

| 属性 | 描述 |
| --- | --- |
| Class | 这个属性是强制性的，并且指定用来创建 bean 的 bean 类 |
| Name | 这个属性指定唯一的 bean 标识符；在基于 XML 的配置元数据中，可以使用 id 和/或 name 属性来指定 bean 标识符 |
| Scope | 这个属性指定由特定的 bean 定义创建的对象的作用域 |
| Constructor arguments | 依赖注入 |
| Properties | 依赖注入 |
| Autowiring mode | 自动装配 |
| Lazy initialization mode | 延迟初始化的 bean ，告诉 IoC 容器在它第一次被请求时创建，而不是在启动时创建一个 bean 实例 |
| Initialization method | 在 bean 的所有必需的属性被容器设置之后，调用回调方法 |
| Destruction method | 当包含该 bean 的容器被销毁时，使用回调方法 |

## 创建Bean对象的三种方式

编写 service 层接口及其实现类作为测试用例：

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
    
    import com.ydx.service.AccountService;
    
    /**
     * 账户的业务层实现类
     */
    public class AccountServiceImpl implements AccountService {
    
        public AccountServiceImpl() {
            System.out.println("AccountServiceImpl对象创建了");
        }
    
        public void saveAccount() {
            System.out.println("service中的saveAccount()方法执行了");
        }
    }
{% endcodeblock %}

{% codeblock BaseTest.java lang:Java %}
    package com.ydx;
    
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
            as.saveAccount();
        }
    }
{% endcodeblock %}

- 使用默认构造方法创建：

  它会根据默认无参构造方法来创建类对象。
  
  > 如果 bean 中没有默认无参构造方法，将会创建失败。

  1. 在配置文件中添加标签
  
    {% codeblock applicationContext.xml lang:XML %}
        <bean id="accountService" class="com.ydx.service.impl.AccountServiceImpl"/>
    {% endcodeblock %}
    
  2. 运行结果
  
    ```console
    AccountServiceImpl对象创建了
    service中的saveAccount()方法执行了
    ```
  
- 使用普通工厂中的方法创建

  使用某个类中的方法创建对象并存入 Spring 容器。

  1. 模拟工厂类
  
    > 实际情况下，该类有可能存在于 jar 包中，我们无法通过修改源码的方式来提供默认构造方法。
  
    {% codeblock InstanceFactory.java lang:Java %}
        package com.ydx.factory;
        
        import com.ydx.service.AccountService;
        import com.ydx.service.impl.AccountServiceImpl;
        
        public class InstanceFactory {
        
          public AccountService getAccountService() {
              return new AccountServiceImpl();
          }
        }
    {% endcodeblock %}
  
  2. 在配置文件中添加标签
  
    {% codeblock applicationContext.xml lang:XML %}
        <bean id="instanceFactory" class="com.ydx.factory.InstanceFactory" />
        <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService" />
    {% endcodeblock %}
    
    > factory-bean 属性指定了工厂类。
    > factroy-method 属性指定使用哪一个方法获取对象。
    
  3. 运行结果
    
    ```console
    AccountServiceImpl对象创建了
    service中的saveAccount()方法执行了
    ```

- 使用工厂中的静态方法创建

  使用某个类中的静态方法创建对象并存入 Spring 容器。
  1. 模拟工厂类
  
    > 实际情况下，该类有可能存在于 jar 包中，我们无法通过修改源码的方式来提供默认构造方法。
  
    {% codeblock StaticFactory.java lang:Java %}
        package com.ydx.factory;
        
        import com.ydx.service.AccountService;
        import com.ydx.service.impl.AccountServiceImpl;
        
        public class StaticFactory {
            public static AccountService getAccountService() {
                return new AccountServiceImpl();
            }
        }
    {% endcodeblock %}
  
  2. 在配置文件中添加标签
  
    {% codeblock applicationContext.xml lang:XML %}
        <bean id="accountService" class="com.ydx.factory.StaticFactory" factory-method="getAccountService"/>
    {% endcodeblock %}
    
  3. 运行结果
    
    ```console
    AccountServiceImpl对象创建了
    service中的saveAccount()方法执行了
    ```

## Bean作用域和生命周期

### Bean作用域

当在 Spring 中定义一个 bean 时，你必须在 **scope 属性**中声明该 bean 的作用域的选项。

Spring 框架支持以下几个作用域：

| 作用域 | 描述 |
| --- | --- |
| singleton | 在 Spring IoC 容器仅存在一个 bean 实例，bean 以单例方式存在，默认值 |
| prototype | 每次从容器中调用 bean 时，都返回一个新的实例，即每次调用 `getBean()` 时，相当于执行 `newXxxBean()` |
| request | 每次 HTTP 请求都会创建一个新的 bean，该作用域仅适用于 `ApplicationContext` 环境 |
| session | 同一个 HTTP Session 共享一个 bean，不同 Session 使用不同的 bean，仅适用于 `ApplicationContext` 环境 |
| application | 将一个 bean 定义作用于 `ServletContext` 的生命周期，该作用域仅适用于 `ApplicationContext` 环境 |
| websocket | 将一个 bean 定义作用于 WebSocket 的生命周期，该作用域仅适用于 `ApplicationContext` 环境 |

> 常用 singleton 和 prototype

### Bean生命周期

在 `bean` 标签中使用 `init-method` 属性来指定在 bean 初始化时执行的方法，使用 `destroy-method` 属性来指定在 bean 销毁时执行的方法，

为了方便观察，在 `AccountServiceImpl` 中添加 `init()` 和 `destroy()` 方法。

{% codeblock AccountServiceImpl.java lang:Java %}
    package com.ydx.service.impl;
    
    import com.ydx.service.AccountService;
    
    /**
     * 账户的业务层实现类
     */
    public class AccountServiceImpl implements AccountService {
    
        public AccountServiceImpl() {
            System.out.println("AccountServiceImpl对象创建了");
        }
    
        public void  saveAccount() {
            System.out.println("service中的saveAccount()方法执行了");
        }
    
        public void  init() {
            System.out.println("AccountServiceImpl对象初始化了");
        }
    
        public void  destroy() {
            System.out.println("AccountServiceImpl对象销毁了");
        }
    }
{% endcodeblock %}

测试类：

{% codeblock BaseTest.java lang:Java %}
    package com.ydx;
    
    import com.ydx.service.AccountService;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    public class BaseTest {
        /**
         * 获取Spring的IoC核心容器，并根据id获取对象
         * @param args
         */
        public static void main(String[] args) {
            //获取核心容器对象
            ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
    
            //根据id获取bean对象
            AccountService as = (AccountService) ac.getBean("accountService");
            as.saveAccount();
    
            //关闭容器
            ac.close();
        }
    }
{% endcodeblock %}

- 单例对象：`scope="singleton"`

  一个应用只有一个对象的实例。它的作用范围就是整个引用。
  
  生命周期：
  - 对象出生：当应用加载，创建容器时，对象就被创建了。
  - 对象存活：只要容器在，对象一直存活。
  - 对象死亡：当应用卸载，销毁容器时，对象就被销毁了。
  
  配置文件：
  
  {% codeblock applicationContext.xml lang:XML %}
      <bean id="accountService" class="com.ydx.service.impl.AccountServiceImpl" scope="singleton" init-method="init" destroy-method="destroy"/>
  {% endcodeblock %}
  
  运行结果：
  
  ```console
  AccountServiceImpl对象创建了
  AccountServiceImpl对象初始化了
  service中的saveAccount()方法执行了
  AccountServiceImpl对象销毁了
  ```
  
  > 容器关闭时，对象也就被销毁了

- 多例对象：`scope="prototype"`

  每次访问对象时，都会重新创建对象实例。
  
  生命周期：
  - 对象出生：当使用对象时，创建新的对象实例。
  - 对象存活：只要对象在使用中，就一直存活。
  - 对象死亡：当对象长时间不用时，被 Java 的垃圾回收器回收了。 
  
  配置文件：
  
  {% codeblock applicationContext.xml lang:XML %}
      <bean id="accountService" class="com.ydx.service.impl.AccountServiceImpl" scope="prototype" init-method="init" destroy-method="destroy"/>
  {% endcodeblock %}
  
  运行结果：
  
  ```console
  AccountServiceImpl对象创建了
  AccountServiceImpl对象初始化了
  service中的saveAccount()方法执行了
  ```

  > 即使关闭了容器，对象还会一直存在

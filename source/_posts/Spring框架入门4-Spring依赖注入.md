---
title: Spring框架入门4. Spring依赖注入
color: '#EBF8FF'
tags:
  - Java
  - Spring
categories: Spring框架入门
abbrlink: a9caf4f0
date: 2019-12-23 12:55:41
---

# Spring框架入门4. Spring依赖注入

## 依赖注入

**依赖注入**（Dependency Injection，缩写为DI）是 Spring 框架核心 IoC 的具体实现。我们的程序在编写时，通过控制反转，把对象的创建交给了 Spring，但是代码中不可能出现没有依赖的情况。

**IoC 解耦只是降低他们的依赖关系，但不会消除**。

在某个类中需要用到其他类的对象，由 Spring 为我们提供，我们只需要在配置文件中说明，这种依赖关系的维护就称之为依赖注入。

例如：我们的业务层仍会调用持久层的方法。 那这种业务层和持久层的依赖关系，在使用 Spring 之后，就让 Spring 来维护了。 简单的说，就是坐等框架把持久层对象传入业务层，而不用我们自己去获取。

## 依赖注入的方式

### 通过构造方法注入

在 `bean` 标签中使用 `constructor-arg` 标签。

`constructor-arg` 标签有五个属性：
- `type`：用于指定参数在构造方法中的数据类型，该数据类型也是要注入数据的数据类型；
- `index`：用于指定参数在构造方法参数列表的索引位置，索引从0开始；
- `name`：用于指定参数在构造函数中的名称，此属性更常用；
- `value`：用于提供基本数据类型和 String 类型的赋值；
- `ref`：用于指定其他 bean 数据类型的赋值，它指的是在 Spring IoC 核心容器中出现过的 bean 对象。 

1. 在实体类 `Account` 中添加构造方法

  {% codeblock Account.java lang:Java %}
    package com.ydx.domain;
    
    import java.util.Date;
    
    public class Account {
        private String name;
        private int age;
        private Date birthday;
        
        public Account(String name, int age, Date birthday) {
            this.name = name;
            this.age = age;
            this.birthday = birthday;
        }
        
        @Override
        public String toString() {
            return "Account{" +
                   "name='" + name + '\'' +
                   ", age=" + age +
                   ", birthday=" + birthday +
                   '}';
        }
    }
  {% endcodeblock %}

2. 配置文件中添加 `bean` 标签

  {% codeblock applicationContext.xml lang:XML %}
      <bean id="account" class="com.ydx.domain.Account" >
          <constructor-arg index="0" value="ydx" />
          <constructor-arg type="int" value="23" />
          <constructor-arg name="birthday" ref="now" />
      </bean>
  
      <!-- 配置一个日期对象 -->
      <bean id="now" class="java.util.Date" />
  {% endcodeblock %}

3. 测试类

  {% codeblock BaseTest.java lang:Java %}
    package com.ydx;
    
    import com.ydx.domain.Account;
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
            Account account = ac.getBean("account", Account.class);
            System.out.println(account.toString());
        }
    }
  {% endcodeblock %}
  
4. 运行结果：

  ```console
    Account{name='ydx', age=23, birthday=Mon Dec 23 14:21:03 CST 2019}
  ```
  
### 通过set方法注入

在 `bean` 标签中使用 `property` 标签。

`property` 标签有三个属性：
- `name`：用于指定参数在构造函数中的名称；
- `value`：用于提供基本数据类型和 String 类型的赋值；
- `ref`：用于指定其他 bean 数据类型的赋值，它指的是在 Spring IoC 核心容器中出现过的 bean 对象。

1. 在实体类 `Account` 中为属性添加 set 方法

  {% codeblock Account.java lang:Java %}
    package com.ydx.domain;
    
    import java.util.Date;
    
    public class Account {
        private String name;
        private int age;
        private Date birthday;
    
        public void setName(String name) {
            this.name = name;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public void setBirthday(Date birthday) {
            this.birthday = birthday;
        }
    
        @Override
        public String toString() {
            return "Account{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    ", birthday=" + birthday +
                    '}';
        }
    }
  {% endcodeblock %}

2. 配置文件中添加 `bean` 标签

  {% codeblock applicationContext.xml lang:XML %}
    <bean id="account" class="com.ydx.domain.Account" >
        <property name="name" value="ydx" />
        <property name="age" value="23" />
        <property name="birthday" ref="now" />
    </bean>
    
    <!-- 配置一个日期对象 -->
    <bean id="now" class="java.util.Date" />
  {% endcodeblock %}

3. 测试类

  {% codeblock BaseTest.java lang:Java %}
    package com.ydx;
    
    import com.ydx.domain.Account;
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
            Account account = ac.getBean("account", Account.class);
            System.out.println(account.toString());
        }
    }
  {% endcodeblock %}
  
4. 运行结果：

  ```console
    Account{name='ydx', age=23, birthday=Mon Dec 23 14:21:03 CST 2019}
  ```
  
## 复杂/集合类型的注入

以下示例采用 set 注入方式

1. 编写 `JavaCollection` 类
  
  {% codeblock JavaCollection.java lang:Java %}
    package com.ydx.domain;
    
    import java.util.*;
    
    public class JavaCollection {
    
        private String[] myStr;
        private List<String> myList;
        private Set<String> mySet;
        private Map<String, String> myMap;
        private Properties myProp;
    
        public void setMyStr(String[] myStr) {
            this.myStr = myStr;
        }
    
        public void setMyList(List<String> myList) {
            this.myList = myList;
        }
    
        public void setMySet(Set<String> mySet) {
            this.mySet = mySet;
        }
    
        public void setMyMap(Map<String, String> myMap) {
            this.myMap = myMap;
        }
    
        public void setMyProp(Properties myProp) {
            this.myProp = myProp;
        }
    
        @Override
        public String toString() {
            return "JavaCollection{" +
                    "\nmyStr=" + Arrays.toString(myStr) +
                    ", \nmyList=" + myList +
                    ", \nmySet=" + mySet +
                    ", \nmyMap=" + myMap +
                    ", \nmyProp=" + myProp +
                    "\n}";
        }
    }
  {% endcodeblock %}

2. 配置文件中添加 `bean` 标签

  {% codeblock applicationContext.xml lang:XML %}
    <bean id="jc" class="com.ydx.domain.JavaCollection" >
        <property name="myStr">
            <array>
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </array>
        </property>

        <property name="myList">
            <list>
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </list>
        </property>

        <property name="mySet">
            <set>
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </set>
        </property>

        <property name="myMap">
            <map>
                <entry key="test1" value="aaa" />
                <entry key="test2">
                    <value>bbb</value>
                </entry>
            </map>
        </property>

        <property name="myProp">
            <props>
                <prop key="test3">ccc</prop>
                <prop key="test4">ddd</prop>
            </props>
        </property>
    </bean>
  {% endcodeblock %}

3. 测试类

  {% codeblock BaseTest.java lang:Java %}
    package com.ydx;
    
    import com.ydx.domain.JavaCollection;
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
            JavaCollection jc = ac.getBean("jc", JavaCollection.class);
            System.out.println(jc.toString());
        }
    }
  {% endcodeblock %}
  
4. 运行结果：

  ```console
    JavaCollection{
    myStr=[aaa, bbb, ccc], 
    myList=[aaa, bbb, ccc], 
    mySet=[aaa, bbb, ccc], 
    myMap={test1=aaa, test2=bbb}, 
    myProp={test4=ddd, test3=ccc}
    }
  ```

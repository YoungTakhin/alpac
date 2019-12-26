---
title: Spring框架入门5. 基于XML的IoC案例
color: '#EBF8FF'
tags:
  - Java
  - Spring
categories: Spring框架入门
abbrlink: '7423130'
date: 2019-12-24 17:53:23
---

# Spring框架入门5. 基于XML的IoC案例

> 本案例结合 DBUtil 包，数据库使用 MySQL 实现查询所有用户的简单功能

## 环境搭建

1. 创建数据库 spring-learn 并导入 SQL 脚本：

  ```sql
    create table account(
        id int primary key auto_increment,
        name varchar(40),
        money float
    )character set utf8 collate utf8_general_ci;
    
    insert into account(name,money) values('aaa',1000);
    insert into account(name,money) values('bbb',1000);
    insert into account(name,money) values('ccc',1000);
  ```

2. Maven 项目 pom.xml 文件配置如下：

  {% codeblock pom.xml lang:XML %}
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>com.ydx</groupId>
        <artifactId>spring-learn</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>jar</packaging>
    
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>5.2.2.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>commons-dbutils</groupId>
                <artifactId>commons-dbutils</artifactId>
                <version>1.7</version>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.17</version>
            </dependency>
            <dependency>
                <groupId>c3p0</groupId>
                <artifactId>c3p0</artifactId>
                <version>0.9.1.2</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
            </dependency>
        </dependencies>
        
    </project>
  {% endcodeblock %}

3. 在 resource 目录下新建数据库配置文件 jdbc.properties，写入数据库配置

  {% codeblock jdbc.properties lang:properties %}
    jdbc.driver=com.mysql.cj.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/spring-learn?serverTimezone=UTC&characterEncoding=utf-8
    jdbc.username=root
    jdbc.password=123456
  {% endcodeblock %}

3. 在 domain 包下创建 Account 实体类对象：

  {% codeblock Account.java lang:Java %}
    package com.ydx.domain;
    
    import java.io.Serializable;
    
    /**
     * 账户的实体类
     */
    public class Account implements Serializable {
    
        private Integer id;
        private String name;
        private Float money;
    
        public Integer getId() {
            return id;
        }
    
        public void setId(Integer id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public Float getMoney() {
            return money;
        }
    
        public void setMoney(Float money) {
            this.money = money;
        }
    
        @Override
        public String toString() {
            return "Account{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    ", money=" + money +
                    '}';
        }
    }
  {% endcodeblock %}

4. 编写 dao 层接口及其实现类

  {% codeblock AccountDao.java lang:Java %}
    package com.ydx.dao;
    
    import com.ydx.domain.Account;
    
    import java.util.List;
    
    /**
    * 账户的持久层接口
    */
    public interface AccountDao {
    
      /**
       * 查询所有
       * @return
       */
      List<Account> findAllAccount();
    }
  {% endcodeblock %}
  
  {% codeblock AccountDaoImpl.java lang:Java %}
    package com.ydx.dao.impl;
    
    import com.ydx.dao.AccountDao;
    import com.ydx.domain.Account;
    import org.apache.commons.dbutils.QueryRunner;
    import org.apache.commons.dbutils.handlers.BeanListHandler;
    
    import java.util.List;
    
    /**
     * 账户的持久层实现类
     */
    public class AccountDaoImpl implements AccountDao {
    
        private QueryRunner runner;
    
        public void setRunner(QueryRunner runner) {
            this.runner = runner;
        }
    
        @Override
        public List<Account> findAllAccount() {
            try{
                return runner.query("select * from account", new BeanListHandler<>(Account.class));
            }catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }
  {% endcodeblock %}
  
5. 编写 service 层接口及其实现类 
  
  {% codeblock AccountService.java lang:Java %}
    package com.ydx.service;
    
    import com.ydx.domain.Account;
    
    import java.util.List;
    
    /**
     * 账户业务层的接口
     */
    public interface AccountService {
    
        /**
         * 查询所有
         * @return
         */
        List<Account> findAllAccount();
    }
  {% endcodeblock %}
    
  {% codeblock AccountServiceImpl.java lang:Java %}
    package com.ydx.service.impl;
    
    import com.ydx.dao.AccountDao;
    import com.ydx.domain.Account;
    import com.ydx.service.AccountService;
    
    import java.util.List;
    
    public class AccountServiceImpl implements AccountService {
    
        private AccountDao accountDao;
    
        public void setAccountDao(AccountDao accountDao) {
            this.accountDao = accountDao;
        }
    
        @Override
        public List<Account> findAllAccount() {
            return accountDao.findAllAccount();
        }
    }
  {% endcodeblock %}
  
6. 在 resource 目录下配置 Spring 上下文

  {% codeblock applicationContext.xml lang:XML %}
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <!-- 加载properties文件 -->
        <bean id="propertyConfigurer" class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
            <property name="locations" value="/jdbc.properties"/>
        </bean>
    
        <!-- 配置数据源 -->
        <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
            <!--连接数据库的必备信息-->
            <property name="driverClass" value="${jdbc.driver}"/>
            <property name="jdbcUrl" value="${jdbc.url}"/>
            <property name="user" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>
    
        <!-- 配置service -->
        <bean id="accountService" class="com.ydx.service.impl.AccountServiceImpl">
            <!-- 注入dao -->
            <property name="accountDao" ref="accountDao"/>
        </bean>
    
        <!-- 配置dao -->
        <bean id="accountDao" class="com.ydx.dao.impl.AccountDaoImpl">
            <!-- 注入QueryRunner -->
            <property name="runner" ref="runner"/>
        </bean>
    
        <!-- 配置QueryRunner -->
        <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
            <!-- 注入数据源 -->
            <constructor-arg name="ds" ref="dataSource"/>
        </bean>
    
    </beans>
  {% endcodeblock %}
  
最终项目目录结构如下：

  ![目录结构](https://s2.ax1x.com/2019/12/25/liYMuD.png)

## 运行测试

1. 编写测试类

  {% codeblock BaseTest.java lang:Java %}
    package com.ydx;
    
    import com.ydx.domain.Account;
    import com.ydx.service.AccountService;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    import java.util.List;
    
    public class BaseTest {
    
        /**
         * 获取Spring的IoC核心容器，并根据id获取对象
         * @param args
         */
        public static void main(String[] args) {
            //获取核心容器对象
            ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
    
            //根据id获取bean对象
            AccountService as = ac.getBean("accountService", AccountService.class);
            List<Account> accounts = as.findAllAccount();
            for(Account account : accounts) {
                System.out.println(account);
            }
        }
    }
  {% endcodeblock %}
  
2. 运行结果

  ```console
    Account{id=1, name='aaa', money=1000.0}
    Account{id=2, name='bbb', money=1000.0}
    Account{id=3, name='ccc', money=1000.0}
  ```

---
title: Spring框架入门6 Spring基于注解开发的配置
color: '#EBF8FF'
tags:
  - Java
  - Spring
categories: Spring框架入门
abbrlink: 3adebc70
date: 2019-12-25 14:17:14
---

# Spring框架入门6 Spring基于注解开发的配置

## 使用XML配置

基于上一案例修改

1. 修改 Spring 上下文配置

  {% codeblock applicationContext.xml lang:XML %}
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            https://www.springframework.org/schema/context/spring-context.xsd">
    
        <!-- 开启注解（声明了context:component-scan时，可省略） -->
        <context:annotation-config/>
    
        <!-- 告知Spring在创建容器时要扫描的包 -->
        <context:component-scan base-package="com.ydx"/>
        <!--配置QueryRunner -->
        <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
            <!--注入数据源 -->
            <constructor-arg name="ds" ref="dataSource"/>
        </bean>
    
        <!-- 加载properties文件 -->
        <context:property-placeholder location="classpath:jdbc.properties"/>
        <!-- 配置数据源 -->
        <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
            <!-- 连接数据库的必备信息 -->
            <property name="driverClass" value="${jdbc.driver}"/>
            <property name="jdbcUrl" value="${jdbc.url}"/>
            <property name="user" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>
    </beans>
  {% endcodeblock %}
  
2. 基于注解的装配与注入

  - 与创建对象有关的注解：

    - `@Component` 注解标记在类上，当使用了基于注解的配置和类路径扫描时，会把当前类对象作为 bean 存入 Spring IoC 容器中。

      -  属性 `value` 用于指定 bean id，默认值为类名首字母大写转小写。

    与 `@Component` 注解的功能完全等同的注解还有以下三种：
    
    - `@Repository`：通常标记在持久层类上。
    - `@Service`：通常标记在业务层类上。
    - `@Controller`：通常标记在表现层类上。

  - 与注入数据有关的注解：
  
    > 使用注解注入时，类的 set 方法可省略
  
    - `@Value`：`@Value` 注解可以标记在类、方法、形参、注解上，用于注入基本类型和 String 类型，还支持 Spring 的 EL 表达式。
    - `@Autowired`：`@Autowired` 注解可以标记在方法、构造方法、形参、属性、注解上，默认情况下，该注解以 byType 方式注入，若有多个 bean， 则又以 byName 方式注入，若还有多个 bean 则抛出异常。
    - `@Qualifier`：`@Qualifier` 注解可以标记在属性、方法、形参、类、注解上，一般与 `@Autowired` 注解一起使用，在自动类型注入基础上，再按照 Bean id 注入。

  使用以上注解修改 service 和 dao 层的实现类：

  {% codeblock accountServiceImpl.java lang:Java %}
    package com.ydx.service.impl;
    
    import com.ydx.dao.AccountDao;
    import com.ydx.domain.Account;
    import com.ydx.service.AccountService;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    
    import java.util.List;
    
    @Service("accountService")
    public class AccountServiceImpl implements AccountService {
    
        @Autowired //自动注入
        private AccountDao accountDao;
    
        @Override
        public List<Account> findAllAccount() {
            return accountDao.findAllAccount();
        }
    }
  {% endcodeblock %}
  
  {% codeblock accountDaoImpl.java lang:Java %}
    package com.ydx.dao.impl;
    
    import com.ydx.dao.AccountDao;
    import com.ydx.domain.Account;
    import org.apache.commons.dbutils.QueryRunner;
    import org.apache.commons.dbutils.handlers.BeanListHandler;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Repository;
    
    import java.util.List;
    
    /**
     * 账户的持久层实现类
     */
    @Repository("accountDao")
    public class AccountDaoImpl implements AccountDao {
    
        @Autowired //自动注入
        private QueryRunner runner;
    
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

## 使用Java配置类配置

基于上一案例修改

1. 新建包 config 作为配置包

2. 新建配置类：

  - 与上下文有关的注解：
    - `@Configuration`：`@Configuration` 注解标记在类上，用于指定当前类为主配置类，当配置类作为 `AnnotationConfigApplicationContext` 对象创建的参数时，该注释可省略。
    - `@ComponentScan`：`@ComponentScan` 注解标记在类上，用于指定 Spring 在创建时要扫描的包。
      - 属性 `value` 指定包的类路径。
    - `@ComponentScans`：可以扫描多个包的类路径，同 `@ComponentScan` 注解。
    - `@Bean`：`@Bean` 注解标记在方法、注解上，用于把当前方法的返回值当作 bean 对象存入 Spring IoC 容器中。
      - 属性 `value` 指定 bean id，默认值为被标记的方法名。
    - `@Import`：`@Import`注解标记在类上，用于导入其他配置类。
      - 属性 `value` 指定其他配置类的字节码。
    - `@PropertySource`：`@PropertySource` 注解标记在类上，用于读取 resouces 下的 properties 文件。
      - 属性 `value` 格式为"classpath:文件名.properties"。
    - `@PropertySources`：可以读取多个 properties 文件，同 `@PropertySource` 注解。
  - 与作用域及生命周期有关的注解：
    - `@Scope`：`@Scope` 注解标记在类、方法上，用于指定 bean 的作用域。
      - 属性 `value` 取值同 `<bean>` 中的 `scope` 属性。
    - `@PostConstruct`：`@PostConstruct` 注解标记在方法上，声明该方法在 bean 创建时立即执行。
    - `@PreDestroy`：`@PreDestroy`  注解标记在方法上，声明该方法在 bean 销毁时执行。

  使用上述标签新建主配置类 SpringConfig.java
  
  {% codeblock SpringConfig.java lang:Java %}
    package config;
    
    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Import;
    import org.springframework.context.annotation.PropertySource;
    
    /**
     * Spring配置类
     */
    @Configuration //指定当前类为主配置类，作为AnnotationConfigApplicationContext的参数时可省略
    @ComponentScan("com.ydx") //指定Spring创建是要扫描的包
    @Import(JdbcConfig.class) //要导入的其他配置文件
    @PropertySource("classpath:jdbc.properties") //指定properties文件位置
    public class SpringConfig {
        
    }
  {% endcodeblock %}

  新建 JDBC 配置类 JdbcConfig.java
  
  {% codeblock JdbcConfig.java lang:Java %}
    package config;
    
    import com.mchange.v2.c3p0.ComboPooledDataSource;
    import org.apache.commons.dbutils.QueryRunner;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Scope;
    
    import javax.sql.DataSource;
    
    /**
     * JDBC配置类
     */
    public class JdbcConfig {
    
        @Value("${jdbc.driver}") //从properties文件注入数据
        private String driver;
    
        @Value("${jdbc.url}")
        private String url;
    
        @Value("${jdbc.username}")
        private String username;
    
        @Value("${jdbc.password}")
        private String password;
    
        /**
         * 加载数据源
         * @return
         */
        @Bean //创建了名字为dataSource的bean，存入IoC容器
        public DataSource dataSource() {
            try {
                ComboPooledDataSource ds = new ComboPooledDataSource();
                ds.setDriverClass(driver);
                ds.setJdbcUrl(url);
                ds.setUser(username);
                ds.setPassword(password);
                return ds;
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    
        /**
         * 创建QueryRunner对象
         * @param dataSource
         * @return
         */
        @Bean(name = "qr") //Spring创建了一个名字叫qr的bean，存入IoC容器
        @Scope("prototype") //多例对象
        public QueryRunner queryRunner(@Qualifier("dataSource") //指定bean id注入
                                                   DataSource dataSource) {
            return new QueryRunner(dataSource);
        }
    }
  {% endcodeblock %}

3. 修改测试类

  {% codeblock JdbcConfig.java lang:Java %}
    package com.ydx;
    
    import com.ydx.domain.Account;
    import com.ydx.service.AccountService;
    import config.SpringConfig;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    import java.util.List;
    
    public class BaseTest {
    
        /**
         * 获取Spring的IoC核心容器，并根据id获取对象
         * @param args
         */
        public static void main(String[] args) {
            //获取核心容器对象，这里使用了AnnotationConfigApplicationContext
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfig.class);
    
            //根据id获取bean对象
            AccountService as = ac.getBean("accountService", AccountService.class);
            List<Account> accounts = as.findAllAccount();
            for(Account account : accounts) {
                System.out.println(account);
            }
        }
    }
  {% endcodeblock %}

---
title: MyBatis框架入门二. MyBatis框架入门
color: '#EBF8FF'
tags:
  - Java
  - MyBatis
categories: MyBatis框架入门
abbrlink: 39266
date: 2019-11-16 15:24:43
---
# MyBatis框架入门二. MyBatis框架入门

## MyBatis概述
MyBatis是用Java编写的持久型框架，封装了JDBC操作的很多细节，使开发者只需要关注SQL语句本身，无需关注注册驱动、创建连接等繁杂过程；

MyBatis使用ORM思想实现了结果集的封装。

### ORM
Obeject Relational Mapping对象关系映射：

把数据库表和实体类的属性对应起来，让我们可以操作实体类就实现操作数据库表

## MyBatis入门示例

### MyBatis环境搭建
1. 创建Maven工程并导入坐标

    pom.xml文件如下配置：
    
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.ydx</groupId>
       <artifactId>MyBatis_test</artifactId>
       <version>1.0-SNAPSHOT</version>
       <!-- 打包方式-->
       <packaging>jar</packaging>
   
       <!-- 依赖列表-->
       <dependencies>
           <!-- MyBatis-->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.2</version>
           </dependency>
           <!-- MySQL-->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>8.0.15</version>
           </dependency>
           <!-- log4j-->
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>1.2.17</version>
           </dependency>
           <!-- JUnit-->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   </project>
   ```
2. 创建实体类和dao的接口，如实体类User，dao接口UserDao
    
   实体类中的属性与数据库表中字段对应，示例如下：
   ```java
   package com.ydx.domain;
   
   import java.io.Serializable;
   import java.util.Date;
   
   public class User implements Serializable {
       private Integer id;
   
       private String username;
   
       private Date birthday;
   
       private String sex;
   
       private String address;
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getUsername() {
           return username;
       }
   
       public void setUsername(String username) {
           this.username = username;
       }
   
       public Date getBirthday() {
           return birthday;
       }
   
       public void setBirthday(Date birthday) {
           this.birthday = birthday;
       }
   
       public String getSex() {
           return sex;
       }
   
       public void setSex(String sex) {
           this.sex = sex;
       }
   
       public String getAddress() {
           return address;
       }
   
       public void setAddress(String address) {
           this.address = address;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "id=" + id +
                   ", username='" + username + '\'' +
                   ", birthday=" + birthday +
                   ", sex='" + sex + '\'' +
                   ", address='" + address + '\'' +
                   '}';
       }
   }
   ```
   
   dao的接口示例如下：
   
   ```java
   package com.ydx.dao;
   
   import com.ydx.domain.User;
   
   import java.util.List;
   
   /**
    * 用户持久层接口
    */
   public interface UserDao {
   
       /**
        * 查询所有操作
        * @return
        */
       List<User> findAll();
   }
   ```

3. 创建MyBatis的主配置文件：SqlMapConfig.xml
   
    主配置文件SqlMapConfig.xml配置示例如下：
   
   ```xml 
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <!-- MyBatis的主配置文件-->
   <configuration>
       <!-- 配置环境-->
       <environments default="mysql">
           <!-- 配置MySQL的环境-->
           <environment id="mysql">
               <!-- 配置事务类型-->
               <transactionManager type="JDBC"/>
               <!-- 配置数据源（连接池）-->
               <dataSource type="POOLED">
                   <!-- 配置连接数据库的4个基本信息。
                        有了它就能创建Connection对象。
                   -->
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=utf8"/>
                   <property name="username" value="用户名"/>
                   <property name="password" value="密码"/>
               </dataSource>
           </environment>
       </environments>
   
       <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件；
            如果使用注解来配置的话，此处应该使用class属性指定被注解的dao全限定类名。
            有了它就有了映射配置信息。
       -->
       <mappers>
           <mapper resource="com/ydx/dao/UserDao.xml"/>
       </mappers>
   </configuration>
   ```

4. 创建映射配置文件：如UserDao.xml

    映射配置文件配置如下：
 
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <!-- namespace中要写全限定类名-->
   <mapper namespace="com.ydx.dao.UserDao">
      <!-- 配置查询所有，id要写方法名称, resultType要写返回类型。
           有了它就有了执行的SQL语句，就可以获取PrepaareSatement
      -->
      <select id="findAll" resultType="com.ydx.domain.User">
          SELECT * FROM user;
      </select>
   </mapper>
   ```


**注意：**
- 创建UserDao.xml和UserDao.Java时名称是为了和我们之前的只是保持一致。在MyBatis中它把持久层的操作接口名称和映射文件也叫做Mapper，所以UserDao.xml和UserMapper是一样的
- 在IDEA中创建目录的时候，它和包是不一样的，包在创建时com.ydx.dao是三级结构，而目录在创建时com.ydx.dao是一级目录
- MyBatis的映射配置文件位置必须和dao接口的包结构相同
- 映射配置文件的mapper标签namespace属性的取值必须是dao接口的全限定类名
- 映射配置文件的操作配置，id属性的取值必须是dao接口的方法名

    当我们遵从了第3、4、5点后，我们之后在开发中就无需再写dao的实现类
    
   
以上读取配置文件用到了dom4j解析xml技术。

### MyBatis的入门示例

#### MyBatis基于xml的入门示例

1. 读取配置文件
    ```java
    InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
    ```
   读取路径时使用相对路径与绝对路径都会存在问题，在实际开发中应该使用以下两种方法：
   1. 使用类加载器，他只能读取类路径的配置文件；
   2. 使用ServletContext对象的getRealPath()，它能得到当前应用部署的绝对路径。
   
2. 创建SqlSessionFactory工厂
    ```java 
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    SqlSessionFactory factory = builder.build(in);
   ```
   创建工厂时MyBatis使用了构建者模式，**构建者模式优点：把对象的创建细节隐藏，是使用者直接调用方法即可拿到对象**。
   
3. 创建SqlSession对象
    ```java 
    SqlSession session = factory.openSession();
   ```
   生产SqlSession使用了工厂模式，**工厂模式优点：解耦（降低类之间的依赖关系）**。
   
4. 创建dao接口的代理对象
    ```java 
    UserDao userdao = session.getMapper(UserDao.class);
   ```
   创建dao接口实现类使用了代理模式，**代理模式优点：不修改源码的基础上对已有方法增强**。
   
5. 执行dao中的方法
    ```java 
    List<User> users = userdao.findAll();
    for (User user : users) {
        System.out.println(user);
    }
   ```
6. 释放资源
    ```java 
    session.close();
    in.close();
   ```

完整代码如下：

```java 
package com.ydx.test;

import com.ydx.dao.UserDao;
import com.ydx.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * MyBatis示例
 */
public class MyBatisTest {

    /**
     * MyBatis示例
     */
    public static void main(String[] args) throws IOException {
        //读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //创建SqlSessionFactiory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
        //使用SqlSession创建dao接口的代理对象
        UserDao userDao = session.getMapper(UserDao.class);
        //使用代理对象执行方法
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
        //释放资源
        session.close();
        in.close();
    }
}

```

**注意：**
- 不要忘记在映射配置文件中告知MyBatis要封装到那个实体类中，配置的方式：指定实体类的全限定类名

#### MyBatis基于注解的入门示例

把UserDao.xml移除，在dao接口的方法上使用@Select注解，并指定SQL语，同时需要在SqlMapConfig.xml中使用mapper配置时，使用class属性指定dao接口的全限定类名。

dao接口中方法代码如下：
```java 
public interface UserDao {

    /**
     * 查询所有操作
     * @return
     */
    @Select("SELECT * FROM user;")
    List<User> findAll();
}
```


SqlMapConfig.xml中配置如下：
```xml
    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件；
        如果使用注解来配置的话，此处应该使用class属性指定被注解的dao全限定类名
    -->
    <mappers>
        <mapper class="com.ydx.dao.UserDao"/>
    </mappers>
```

#### MyBatis支持dao实现类

MyBatis也是支持写dao实现类的，但在实际开发中，为了简便，不管使用xml还是注解类型，都是采用不写dao实现类的方式。

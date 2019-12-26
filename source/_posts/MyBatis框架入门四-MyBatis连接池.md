---
title: MyBatis框架入门四. MyBatis连接池
color: '#EBF8FF'
tags:
  - Java
  - MyBatis
categories: MyBatis框架入门
abbrlink: 26164
date: 2019-11-19 14:10:08
---
# MyBatis框架入门四. MyBatis连接池

## 连接池

在实际开发中使用连接池可以减少我们获取连接所获取的时间。

## MyBatis中的连接池

配置位置：主配置文件sqlMapConfig.xml中的dataSource标签的type属性。

MyBatis连接池提供了3中配置方式（type属性的三种取值）：

- POOLED：
    
    采用传统的javax.sql.DataSource规范中的连接池，MyBatis中有针对规范的实现。
    
    如果将type属性设置成POOLED，MyBatis会创建一个数据库连接池，连接池中的一个连接将会被用作数据操作；一旦数据库操作完成，MyBatis会将此连接返回给连接池。
    
    在开发或测试环境中，经常使用此种方式
    
- UNPOOLED：

    采用传统的连接方式，也是实现了javax.sql.DataSource规范中的接口，但是并没有使用池的思想。
    
    如果将type属性设置成POOLED，MyBatis会为每一个数据库操作创建一个新的连接，并关闭它。
    
    该方式适用于只有小规模数量并发用户的简单应用程序上。
    
- JNDI：

    Java Naming and Directory Interface。是SUN公司推出的一套规范，属于JavaEE技术之一。目的是模仿windows系统中的注册表。JNDI采用服务器提供的JNDI技术实现，来获取DataSource对象，不同服务器所能拿到的DataSource是不一样的。
 
    在生产环境中优先考虑这种方式。
    
    **注意：如果不是web或Maven war工程，则不能使用。**

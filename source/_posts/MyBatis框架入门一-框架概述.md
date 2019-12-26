---
title: MyBatis框架入门一. 框架概述
color: '#EBF8FF'
tags:
  - Java
  - MyBatis
categories: MyBatis框架入门
abbrlink: 43834
date: 2019-11-16 15:05:20
---
# MyBatis框架入门一. 框架概述

## 什么是框架？

框架（Framework）是整个或部分系统的可重用设计，表现为一组抽象构件及构件实例间交互的方法；另一种定义认为，框架是可被应用开发者定制的应用骨架。前者是从应用方面而后者是从目的方面给出的定义。 

简而言之，框架是软件开发中的一套解决方案，不同框架解决不通问题。

使用框架的好处：框架封装了很多细节，使开发者可使用极简的方式实现功能，提高开发效率。

## MVC三层架构

- 表现层（User Interface layer, UIL）：展示数据
- 业务层（Business Logic Layer, BLL）：处理业务需求
- 持久层（Data access layer, DAL）：数据库交互

![三层架构](https://s2.ax1x.com/2019/11/16/M0OJk4.png)

## 持久层技术解决方案

### JDBC技术

- Connection
- PreparedStatement
- ResultSet

### Spring的JdbcTemplate
Spring中对JDBC的简单封装

### Apache的DBUtils
Apache中对JDBC的简单封装

以上这些都不是框架：

JDBC是规范，Spring的JdbcTemplate和Apache的DBUtils都只是工具类

## JDBC代码示例

如下代码使用了 JDBC 的原始方法（未经封装）实现查询数据库表记录的操作。 

```Java
public static void main(String[] args) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;
    try {
        //加载数据库驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //通过驱动管理类获取数据库链接
        String url = "jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&characterEncoding=utf-8";
        String username = "用户名";
        String password = "密码";
        connection = DriverManager.getConnection(url, username, password);
        //定义SQL语句，?表示占位符
        String sql = "select * from user where username = ?";
        //获取预处理statement
        preparedStatement = connection.prepareStatement(sql);
        //设置参数，第一个参数为SQL语句中参数的序号（从1开始），第二个参数为设置的参数值
        preparedStatement.setString(1, "杨得轩");
        //向数据库发出SQL执行查询，查询出结果集
        resultSet = preparedStatement.executeQuery();
        //遍历查询结果集
        while (resultSet.next()) {
            System.out.println(resultSet.getString("id") + "\t" + resultSet.getString("username"));
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        //释放资源
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 原始JDBC缺点

- 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。
- SQL 语句在代码中硬编码，造成代码不易维护，实际应用 SQL 变化的可能较大，SQL 变动需要改变 Java 代码。 
- 使用 preparedStatement 向占有位符号传参数存在硬编码，因为 SQL 语句的 where 条件不一定，可能多也可能少，修改 SQL 还要修改代码，系统不易维护。
- 对结果集解析存在硬编码(查询列名)，SQL变化导致解析代码变化，系统不易维护，如果能将数据库记 录封装成 POJO 对象解析比较方便。

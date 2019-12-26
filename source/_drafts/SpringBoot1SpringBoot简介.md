---
title: Spring Boot1 Spring Boot简介
tags:
  - Java
  - Spring Boot
categories: Spring Boot
abbrlink: 25934
date: 2019-11-29 10:09:46
---

# Spring Boot1 Spring Boot简介

## Spring Boot

Spring Boot 是 Spring 系列框架中的全新框架，它用来简化 Spring 应用程序的创建和开发过程，可以简单的创建和运行独立的、基于 Spring 框架的生产级应用程序。
绝大多数的Spring Boot应用程序只需要很少的Spring配置。

Spring Boot 可以通过使用 java -jar 或更传统的 war 部署启动来创建 Java 应用程序。Spring Boot 还提供了一个运行“Spring脚本”的命令行工具。

## Spring Boot特性

- 能够快速创建基于Spring框架的应用程序；
- 开箱即用；
- 提供对大型项目类（如嵌入式服务器、安全性、度量标准、健康检查和外部化配置）通用的一系列非功能特性；
- 完全不需要代码生成，也不需要XML配置。

## Spring Boot四大核心

- 自动装配：针对 Spring 应用程序和常见的应用功能，Spring Boot 能自动提供相关配置；
- 起步依赖：对常用依赖进行再次封装，方便引入，简化了 pom.xml 配置，并将依赖管理交给了 Spring Boot，无需关注不同依赖的不同版本是否存在冲突的问题；
- 健康检查：Spring Boot Actuator，可以帮助开发者监控和管理 Spring Boot 应用；
- 命令行界面：Spring Boot 的可选特性，主要针对 Groovy 语言使用。

## 系统要求

Spring Boot 2.2.1.RELEASE 需要 Java 8，并且与 Java 13（包含）兼容，还需要 Spring Framework 5.2.1.RELEASE 或更高版本。

Spring Boot 为以下构建工具提供了明确的构建支持：

| 构建工具 | 版本 |
| :---: | :---: |
| Maven | 3.3+ |
| Gradle | 5.x (同时支持 4.10 但已弃用) |

Spring Boot 支持以下 Servlet 容器：

| 容器 | 版本 |
| :---: | :---: |
| Tomcat 9.0 | 4.0 |
| Jetty 9.4 | 3.1 |
| Undertow 2.0 | 4.0 |

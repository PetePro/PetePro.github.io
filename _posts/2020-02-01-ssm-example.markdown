---
layout: post
title:  "【Java】SSM简单示例"
crawlertitle: "SSM简单示例"
summary: "Spring + SpringMVC + Mybatis"
date:   2020-02-01 09:00:00 +0800
categories: posts
tags: 'CSSE'
author: xusc
---

SSM简单实现

工具：Eclipse Oxygen
环境：JDK 1.8 + Maven 3.6

项目结构一览
```
UserDemo
├── src/main/java
|   ├── com.xusc.controller
|   |   └── UserController.java
|   ├── com.xusc.mapper
|   |   ├── UserMapper.java
|   |   └── UserMapper.xml
|   ├── com.xusc.pojo
|   |   └── User.java
|   ├── com.xusc.service
|   |   └── IUserService.java
|   └── com.xusc.service.impl
|       └── UserService.java
├── src/main/rescources
|   ├── generatorConfig.xml
|   ├── jdbc.properties
|   ├── log4j.properties
|   └── applicationContext.xml
├── src/test/java
├── src/main/webapp
|   ├── WEB-INF
|   |   └── web.xml
|   └── index.jsp
├── target
└── pom.xml
```

### 项目创建与配置
1. 新建Maven项目，选择Web-app
2. 配置pom.xml
    ```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.xusc</groupId>
        <artifactId>UserDemo</artifactId>
        <packaging>war</packaging>
        <version>0.0.1-SNAPSHOT</version>
        <name>UserDemo Maven Webapp</name>
        <url>http://maven.apache.org</url>

        <properties>
            <!-- spring版本号 -->
            <spring.version>4.0.2.RELEASE</spring.version>
            <!-- mybatis版本号 -->
            <mybatis.version>3.2.6</mybatis.version>
            <!-- log4j日志文件管理包版本 -->
            <slf4j.version>1.7.7</slf4j.version>
            <log4j.version>1.2.17</log4j.version>
        </properties>

        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.11</version>
                <!-- 表示开发的时候引入，发布的时候不会加载此包 -->
                <scope>test</scope>
            </dependency>
            <!-- spring核心包 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-oxm</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-tx</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aop</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- mybatis核心包 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>
            <!-- mybatis/spring包 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>1.2.2</version>
            </dependency>
            <!-- 导入java ee jar 包 -->
            <dependency>
                <groupId>javax</groupId>
                <artifactId>javaee-api</artifactId>
                <version>7.0</version>
            </dependency>
            <!-- 导入Mysql数据库链接jar包 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.41</version>
            </dependency>
            <!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->
            <dependency>
                <groupId>commons-dbcp</groupId>
                <artifactId>commons-dbcp</artifactId>
                <version>1.2.2</version>
            </dependency>
            <!-- JSTL标签类 -->
            <dependency>
                <groupId>jstl</groupId>
                <artifactId>jstl</artifactId>
                <version>1.2</version>
            </dependency>
            <!-- 日志文件管理包 -->
            <!-- log start -->
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <!-- 格式化对象，方便输出日志 -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
                <version>1.1.41</version>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>${slf4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>${slf4j.version}</version>
            </dependency>
            <!-- log end -->
            <!-- 映入JSON -->
            <dependency>
                <groupId>org.codehaus.jackson</groupId>
                <artifactId>jackson-mapper-asl</artifactId>
                <version>1.9.13</version>
            </dependency>
            <!-- 上传组件包 -->
            <dependency>
                <groupId>commons-fileupload</groupId>
                <artifactId>commons-fileupload</artifactId>
                <version>1.3.1</version>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.4</version>
            </dependency>
            <dependency>
                <groupId>commons-codec</groupId>
                <artifactId>commons-codec</artifactId>
                <version>1.9</version>
            </dependency>
            <!-- MyBatis Generator自动创建代码 -->
            <dependency>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-core</artifactId>
                <version>1.3.2</version>
            </dependency>
        </dependencies>

        <build>
            <finalName>UserDemo</finalName>
            <plugins>
                <plugin>
                    <groupId>org.mybatis.generator</groupId>
                    <artifactId>mybatis-generator-maven-plugin</artifactId>
                    <version>1.3.2</version>
                    <configuration>
                        <!-- mybatis用于生成代码的配置文件 -->
                        <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                        <verbose>true</verbose>
                        <overwrite>true</overwrite>
                    </configuration>
                </plugin>
            </plugins>
        </build>

    </project>
    ```
3. 

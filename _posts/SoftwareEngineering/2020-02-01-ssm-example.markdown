---
layout: post
title:  "【Java】SSM框架 总结 02 简单示例"
crawlertitle: "SSM简单示例"
summary: "Spring + SpringMVC + Mybatis"
date:   2020-02-01 10:00:00 +0800
categories: posts
tags: 'SE'
author: xusc
bg: "SE.jpg"
---

`SSM (Spring + SpringMVC + MyBatis) `框架集由 Spring、MyBatis 两个开源框架整合而成，常作为数据源较简单的 Web 项目框架。

- JDK 1.8
- Maven 3.6
- MySQL 5.7
- tomcat 9.0

项目结构一览
```
UserDemo
├── src/main/java
|   ├── com.xusc.controller    .......... controller
|   |   └── UserController.java
|   ├── com.xusc.mapper    .............. mapper，又称DAO，数据访问层，与数据库交互
|   |   ├── UserMapper.java    .......... mapper接口，不用实现
|   |   └── UserMapper.xml    ........... 接口配置文件
|   ├── com.xusc.pojo    ................ pojo，实体类
|   |   └── User.java
|   ├── com.xusc.service    ............. service，业务逻辑层
|   |   └── IUserService.java
|   └── com.xusc.service.impl    ........ 实现业务接口
|       └── UserService.java
├── src/main/rescources
|   ├── generatorConfig.xml    .......... Mybatis自动生成包的配置文件
|   ├── jdbc.properties    .............. jdbc属性文件
|   ├── log4j.properties    ............. log4j属性文件
|   ├── spring-mybatis.xml    ........... 整合Spring与Mybatis的配置文件
|   └── spring-mvc.xml    ............... 整合Spring与MVC的配置文件
├── src/test/java
├── src/main/webapp
|   ├── WEB-INF
|   |   ├── jsp    ...................... 存放jsp页面
|   |   |   └── showUser.jsp
|   |   └── web.xml    .................. 前端配置文件
|   └── index.jsp
├── target
└── pom.xml    .......................... 整个项目的配置文件
```

### 项目创建与配置
1. 新建Maven项目，选择Web-app
2. 配置Maven项目，修改pom.xml
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
3. 整合Spring与Mybatis
   1. 建立jdbc属性文件
        ```
        driver=com.mysql.jdbc.Driver
        url=jdbc:mysql://127.0.0.1:3306/ssm
        username=root
        password=root
        #定义初始连接数
        initialSize=0
        #定义最大连接数
        maxActive=20
        #定义最大空闲
        maxIdle=20
        #定义最小空闲
        minIdle=1
        #定义最长等待时间
        maxWait=60000
        ```
   2. 建立log4j属性文件：使用日志来输出信息
        ```
        #定义LOG输出级别
        log4j.rootLogger=INFO,Console,File
        #定义日志输出目的地为控制台
        log4j.appender.Console=org.apache.log4j.ConsoleAppender
        log4j.appender.Console.Target=System.out
        #可以灵活地指定日志输出格式，下面一行是指定具体的格式
        log4j.appender.Console.layout = org.apache.log4j.PatternLayout
        log4j.appender.Console.layout.ConversionPattern=[%c] - %m%n
        #文件大小到达指定尺寸的时候产生一个新的文件
        log4j.appender.File = org.apache.log4j.RollingFileAppender
        #指定输出目录
        log4j.appender.File.File = logs/ssm.log
        #定义文件最大大小
        log4j.appender.File.MaxFileSize = 10MB
        # 输出所以日志，如果换成DEBUG表示输出DEBUG以上级别日志
        log4j.appender.File.Threshold = ALL
        log4j.appender.File.layout = org.apache.log4j.PatternLayout
        log4j.appender.File.layout.ConversionPattern =[%p] [%d{yyyy-MM-dd HH\:mm\:ss}][%c]%m%n
        ```
   3. 建立spring-mybatis.xml配置文件：自动扫描，自动注入，配置数据库
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:tx="http://www.springframework.org/schema/tx" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
            <!-- 自动扫描 -->
            <context:component-scan base-package="com.xusc" />
            <!-- 引入配置文件 -->
            <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
                <property name="location" value="classpath:jdbc.properties" />
            </bean>
            <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
                <property name="driverClassName" value="${driver}" />
                <property name="url" value="${url}" />
                <property name="username" value="${username}" />
                <property name="password" value="${password}" />
                <!-- 初始化连接大小 -->
                <property name="initialSize" value="${initialSize}"></property>
                <!-- 连接池最大数量 -->
                <property name="maxActive" value="${maxActive}"></property>
                <!-- 连接池最大空闲 -->
                <property name="maxIdle" value="${maxIdle}"></property>
                <!-- 连接池最小空闲 -->
                <property name="minIdle" value="${minIdle}"></property>
                <!-- 获取连接最大等待时间 -->
                <property name="maxWait" value="${maxWait}"></property>
            </bean>
            <!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->
            <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <property name="dataSource" ref="dataSource" />
                <!-- 自动扫描mapping.xml文件 -->
                <property name="mapperLocations" value="classpath:com/xusc/mapper/*.xml"></property>
            </bean>
            <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
            <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
                <property name="basePackage" value="com.xusc.mapper" />
                <property name="sqlSessionFactoryBeanName"
                    value="sqlSessionFactory"></property>
            </bean>
            <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
            <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
                <property name="dataSource" ref="dataSource" />
            </bean>
            <!-- 配置基于注解的声明式事务 -->
            <tx:annotation-driven transaction-manager="transactionManager" />
        </beans>
        ```
   4. 建立数据库并插入相关数据
        ```sql
        CREATE TABLE `user_t` (
        `id` int(11) NOT NULL AUTO_INCREMENT,
        `user_name` varchar(40) NOT NULL,
        `password` varchar(255) NOT NULL,
        `age` int(4) NOT NULL,
        PRIMARY KEY (`id`)
        ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

        insert  into `user_t`(`id`,`user_name`,`password`,`age`) values (1,'abc','abc',24);
        ```
   5. 利用MyBatis Generator自动创建代码
      1. 建立generatorConfig.xml配置文件
            ```xml
            <?xml version="1.0" encoding="UTF-8"?>  
            <!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
            <generatorConfiguration>
                <!-- 数据库驱动 -->
                <classPathEntry
                    location="mysql-connector-java-5.1.41.jar" />
                <context id="DB2Tables" targetRuntime="MyBatis3">
                    <commentGenerator>
                        <property name="suppressDate" value="true" />
                        <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
                        <property name="suppressAllComments" value="true" />
                    </commentGenerator>
                    <!--数据库链接URL，用户名、密码 -->
                    <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/ssm" userId="root" password="root">
                    </jdbcConnection>
                    <javaTypeResolver>
                        <property name="forceBigDecimals" value="false" />
                    </javaTypeResolver>
                    <!-- 生成模型的包名和位置 -->
                    <javaModelGenerator targetPackage="com.xusc.pojo" targetProject="src/main/java">
                        <property name="enableSubPackages" value="true" />
                        <property name="trimStrings" value="true" />
                    </javaModelGenerator>
                    <!-- 生成映射文件的包名和位置 -->
                    <sqlMapGenerator targetPackage="com.xusc.mapper" targetProject="src/main/java">
                        <property name="enableSubPackages" value="true" />
                    </sqlMapGenerator>
                    <!-- 生成DAO的包名和位置 -->
                    <javaClientGenerator type="XMLMAPPER" targetPackage="com.xusc.mapper" targetProject="src/main/java">
                        <property name="enableSubPackages" value="true" />
                    </javaClientGenerator>
                    <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名 -->
                    <table tableName="user_t" domainObjectName="User" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false">
                    </table>
                </context>
            </generatorConfiguration>  
            ```
      2. 右键项目选择`maven build...`，并配置Goals为`mybatis-generator:generate`。成功后会自动生成com.xusc.mapper、com.xusc.pojo两个包以及包下的类、接口和配置文件。
   6. 编写业务逻辑层代码
      - IUserService.java  
        ```java
        package com.xusc.service;
        import com.xusc.pojo.User;
        public interface IUserService {
            public User getUserById(int userId);
        }
        ```
      - UserService.java  
        ```java
        package com.xusc.service.impl;
        import javax.annotation.Resource;
        import org.springframework.stereotype.Service;
        import com.xusc.mapper.UserMapper;
        import com.xusc.pojo.User;
        import com.xusc.service.IUserService;
        @Service("userService")
        public class UserService implements IUserService {
            @Resource
            private UserMapper userMapper;
            public User getUserById(int userId) {
                return this.userMapper.selectByPrimaryKey(userId);
            }
        }
        ```
   7. 至此Spring与Mybatis整合完成，可以在整合MVC前在src/test/java中建立测试文件测试这部分的代码。
4. 整合Spring与MVC
   1. 建立Spring-mvc.xml配置文件：自动扫描控制器，视图模式，注解的启动
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
            <!-- 自动扫描该包，使SpringMVC认为包下用了@controller注解的类是控制器 -->
            <context:component-scan base-package="com.xusc.controller" />
            <!--避免IE执行AJAX时，返回JSON出现下载文件 -->
            <bean id="mappingJacksonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
            <!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->
            <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
                <property name="messageConverters">
                    <list>
                        <ref bean="mappingJacksonHttpMessageConverter" />	<!-- JSON转换器 -->
                    </list>
                </property>
            </bean>
            <!-- 定义跳转的文件的前后缀 ，视图模式配置-->
            <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                <!-- 这里的配置我的理解是自动给后面action的方法return的字符串加上前缀和后缀，变成一个 可用的url地址 -->
                <property name="prefix" value="/WEB-INF/jsp/" />
                <property name="suffix" value=".jsp" />
            </bean>
            <!-- 配置文件上传，如果没有使用文件上传可以不用配置，当然如果不配，那么配置文件中也不必引入上传组件包 -->
            <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">  
                <!-- 默认编码 -->
                <property name="defaultEncoding" value="utf-8" />  
                <!-- 文件大小最大值 -->
                <property name="maxUploadSize" value="10485760000" />  
                <!-- 内存中的最大值 -->
                <property name="maxInMemorySize" value="40960" />  
            </bean> 
        </beans>
        ```
   2. 修改web.xml配置文件
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">
            <display-name>Archetype Created Web Application</display-name>
            <!-- Spring和mybatis的配置文件 -->
            <context-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:spring-mybatis.xml</param-value>
            </context-param>
            <!-- 编码过滤器 -->
            <filter>
                <filter-name>encodingFilter</filter-name>
                <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
                <async-supported>true</async-supported>
                <init-param>
                    <param-name>encoding</param-name>
                    <param-value>UTF-8</param-value>
                </init-param>
            </filter>
            <filter-mapping>
                <filter-name>encodingFilter</filter-name>
                <url-pattern>/*</url-pattern>
            </filter-mapping>
            <!-- Spring监听器 -->
            <listener>
                <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
            </listener>
            <!-- 防止Spring内存溢出监听器 -->
            <listener>
                <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
            </listener>
            <!-- Spring MVC servlet -->
            <servlet>
                <servlet-name>SpringMVC</servlet-name>
                <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
                <init-param>
                    <param-name>contextConfigLocation</param-name>
                    <param-value>classpath:spring-mvc.xml</param-value>
                </init-param>
                <load-on-startup>1</load-on-startup>
                <async-supported>true</async-supported>
            </servlet>
            <servlet-mapping>
                <servlet-name>SpringMVC</servlet-name>
                <!-- 此处可以可以配置成*.do，对应struts的后缀习惯 -->
                <url-pattern>/</url-pattern>
            </servlet-mapping>
            <welcome-file-list>
                <welcome-file>/index.jsp</welcome-file>
            </welcome-file-list>
        </web-app>
        ```
   3. 编写控制层代码
      - UserController.java  
        ```java
        package com.xusc.controller;
        import javax.annotation.Resource;
        import javax.servlet.http.HttpServletRequest;
        import org.springframework.stereotype.Controller;
        import org.springframework.ui.Model;
        import org.springframework.web.bind.annotation.RequestMapping;
        import com.xusc.pojo.User;
        import com.xusc.service.IUserService;
        @Controller
        @RequestMapping("/user")
        public class UserController {
            @Resource
            private IUserService userService;
            @RequestMapping("/showUser")
            public String toIndex(HttpServletRequest request, Model model) {
                int userId = Integer.parseInt(request.getParameter("id"));
                User user = this.userService.getUserById(userId);
                model.addAttribute("user", user);
                return "showUser";
            }
        }
        ```
   4. 编写jsp代码
      - showUser.jsp  
        ```html
        <%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="utf-8"%>
        <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
        <html>
            <head>
                <title>SSM测试</title>
            </head>
            <body>
                ${user.userName}
            </body>
        </html>
        ```
   5. 至此Spring与MVC整合完成，可以在部署项目前在src/test/java中建立测试文件测试这部分的代码。
5. 部署项目至tomcat，在浏览器中输入`localhost:8080/UserDemo/user/showUser?id=1`访问。

本文整合自https://www.cnblogs.com/jay36/p/7762448.html
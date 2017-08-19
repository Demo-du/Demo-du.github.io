---
layout: post
title: maven搭建SSM项目教程 
categories: JavaEE maven
description: 
keywords: 
---

今天用maven尝试了一下搭建SSM的项目，遇到了一些问题并得到了解决。本文以我之前做的一个南航人事管理系统项目（现用maven构建类一遍）为例，下面是具体操作步骤：


## 一、准备环境：

首先要装好maven，jdk，tomcat，这里就不细说了。

## 二、配置Maven、jdk

打开myeclipse，按照下面步骤进行：

1、Window——>Preferences——>Maven——>install设置自己的maven安装路径，然后设置自己的Settings

2、Window——>Preferences——>Java——>Installed JREs——>Add

(/images/posts/819/1.png)

## 三、新建Maven项目：

进入Myeclipse,选择File-New Project-web project

(/images/posts/819/2.jpg)

然后一直点下去，将Group Id，Artifact Id，version等设置好。

然后，项目就搭建好了。之后，我们还要进行一些设置：

右击项目，选择Properties进行一些配置：

(/images/posts/819/3.png)


这样，maven的javaweb项目构建好了，下面我们进行整合搭建SSM（spring MVC + Spring + Mybatis），首先对pox.xml配置依赖的内容：


	    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
	      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">    
		<modelVersion>4.0.0</modelVersion>    
		<groupId>com.ssm</groupId>    
		<artifactId>Maven_Project</artifactId>    
		<packaging>war</packaging>    
		<version>0.0.1-SNAPSHOT</version>    
		<name>Maven_Project Maven Webapp</name>    
		<url>http://maven.apache.org</url>    
		  
		<!-- 用来设置版本号 -->    
		<properties>    
		    <srping.version>4.0.2.RELEASE</srping.version>    
		    <mybatis.version>3.2.8</mybatis.version>    
		    <slf4j.version>1.7.12</slf4j.version>    
		    <log4j.version>1.2.17</log4j.version>    
		</properties>    
		<!-- 用到的jar包 -->    
		<dependencies>    
		    <!-- 单元测试 -->    
		    <dependency>    
		        <groupId>junit</groupId>    
		        <artifactId>junit</artifactId>    
		        <version>4.11</version>    
		        <!-- 表示开发的时候引入，发布的时候不会加载此包 -->      
		        <scope>test</scope>    
		    </dependency>    
		    <!-- java ee包 -->    
		    <dependency>    
		        <groupId>javax</groupId>    
		        <artifactId>javaee-api</artifactId>    
		        <version>7.0</version>    
		    </dependency>    
		    <!-- spring框架包 start -->    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-test</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-core</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-oxm</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-tx</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-jdbc</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-aop</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-context</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-context-support</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-expression</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-orm</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-web</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-webmvc</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <!-- spring框架包 end -->    
		    <!-- mybatis框架包 start -->    
		    <dependency>    
		        <groupId>org.mybatis</groupId>    
		        <artifactId>mybatis</artifactId>    
		        <version>${mybatis.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.mybatis</groupId>    
		        <artifactId>mybatis-spring</artifactId>    
		        <version>1.2.2</version>    
		    </dependency>    
		    <!-- mybatis框架包 end -->    
		    <!-- 数据库驱动 -->    
		    <dependency>    
		        <groupId>mysql</groupId>    
		        <artifactId>mysql-connector-java</artifactId>    
		        <version>5.1.35</version>    
		    </dependency>    
		    <!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->    
		    <dependency>    
		        <groupId>commons-dbcp</groupId>    
		        <artifactId>commons-dbcp</artifactId>    
		        <version>1.4</version>    
		    </dependency>    
		    <!-- jstl标签类 -->    
		    <dependency>    
		        <groupId>jstl</groupId>    
		        <artifactId>jstl</artifactId>    
		        <version>1.2</version>    
		    </dependency>    
		    <!-- log start -->    
		    <dependency>    
		        <groupId>log4j</groupId>    
		        <artifactId>log4j</artifactId>    
		        <version>${log4j.version}</version>    
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
		    <!-- log END -->    
		    <!-- Json  -->    
		    <!-- 格式化对象，方便输出日志 -->    
		    <dependency>    
		        <groupId>com.alibaba</groupId>    
		        <artifactId>fastjson</artifactId>    
		        <version>1.2.6</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.codehaus.jackson</groupId>    
		        <artifactId>jackson-mapper-asl</artifactId>    
		        <version>1.9.13</version>    
		    </dependency>    
		    <!-- 上传组件包 start -->    
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
		        <version>1.10</version>    
		    </dependency>    
		    <!-- 上传组件包 end -->    
		</dependencies>    
		  
		<build>    
		  <finalName>Maven_Project</finalName>    
		</build>    
	    </project>    


	    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
	      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">    
		<modelVersion>4.0.0</modelVersion>    
		<groupId>com.ssm</groupId>    
		<artifactId>Maven_Project</artifactId>    
		<packaging>war</packaging>    
		<version>0.0.1-SNAPSHOT</version>    
		<name>Maven_Project Maven Webapp</name>    
		<url>http://maven.apache.org</url>    
		  
		<!-- 用来设置版本号 -->    
		<properties>    
		    <srping.version>4.0.2.RELEASE</srping.version>    
		    <mybatis.version>3.2.8</mybatis.version>    
		    <slf4j.version>1.7.12</slf4j.version>    
		    <log4j.version>1.2.17</log4j.version>    
		</properties>    
		<!-- 用到的jar包 -->    
		<dependencies>    
		    <!-- 单元测试 -->    
		    <dependency>    
		        <groupId>junit</groupId>    
		        <artifactId>junit</artifactId>    
		        <version>4.11</version>    
		        <!-- 表示开发的时候引入，发布的时候不会加载此包 -->      
		        <scope>test</scope>    
		    </dependency>    
		    <!-- java ee包 -->    
		    <dependency>    
		        <groupId>javax</groupId>    
		        <artifactId>javaee-api</artifactId>    
		        <version>7.0</version>    
		    </dependency>    
		    <!-- spring框架包 start -->    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-test</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-core</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-oxm</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-tx</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-jdbc</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-aop</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-context</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-context-support</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-expression</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-orm</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-web</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.springframework</groupId>    
		        <artifactId>spring-webmvc</artifactId>    
		        <version>${srping.version}</version>    
		    </dependency>    
		    <!-- spring框架包 end -->    
		    <!-- mybatis框架包 start -->    
		    <dependency>    
		        <groupId>org.mybatis</groupId>    
		        <artifactId>mybatis</artifactId>    
		        <version>${mybatis.version}</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.mybatis</groupId>    
		        <artifactId>mybatis-spring</artifactId>    
		        <version>1.2.2</version>    
		    </dependency>    
		    <!-- mybatis框架包 end -->    
		    <!-- 数据库驱动 -->    
		    <dependency>    
		        <groupId>mysql</groupId>    
		        <artifactId>mysql-connector-java</artifactId>    
		        <version>5.1.35</version>    
		    </dependency>    
		    <!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->    
		    <dependency>    
		        <groupId>commons-dbcp</groupId>    
		        <artifactId>commons-dbcp</artifactId>    
		        <version>1.4</version>    
		    </dependency>    
		    <!-- jstl标签类 -->    
		    <dependency>    
		        <groupId>jstl</groupId>    
		        <artifactId>jstl</artifactId>    
		        <version>1.2</version>    
		    </dependency>    
		    <!-- log start -->    
		    <dependency>    
		        <groupId>log4j</groupId>    
		        <artifactId>log4j</artifactId>    
		        <version>${log4j.version}</version>    
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
		    <!-- log END -->    
		    <!-- Json  -->    
		    <!-- 格式化对象，方便输出日志 -->    
		    <dependency>    
		        <groupId>com.alibaba</groupId>    
		        <artifactId>fastjson</artifactId>    
		        <version>1.2.6</version>    
		    </dependency>    
		    <dependency>    
		        <groupId>org.codehaus.jackson</groupId>    
		        <artifactId>jackson-mapper-asl</artifactId>    
		        <version>1.9.13</version>    
		    </dependency>    
		    <!-- 上传组件包 start -->    
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
		        <version>1.10</version>    
		    </dependency>    
		    <!-- 上传组件包 end -->    
		</dependencies>    
		  
		<build>    
		  <finalName>Maven_Project</finalName>    
		</build>    
	    </project>    

在src/main/webapp下添加配置文件：applicationContext.xml:

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans" 
		xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:p="http://www.springframework.org/schema/p"
		xmlns:context="http://www.springframework.org/schema/context"
		xmlns:mvc="http://www.springframework.org/schema/mvc"
		xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="http://www.springframework.org/schema/beans 
					    http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
					    http://www.springframework.org/schema/context
					    http://www.springframework.org/schema/context/spring-context-4.2.xsd
					    http://www.springframework.org/schema/mvc
					    http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
					    http://www.springframework.org/schema/tx
					    http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
					    http://mybatis.org/schema/mybatis-spring 
					    http://mybatis.org/schema/mybatis-spring.xsd ">
				      
		 <!-- mybatis:scan会扫描org.djj.dao包里的所有接口当作Spring的bean配置，之后可以进行依赖注入-->  
	    <mybatis:scan base-package="org.djj.hrm.dao"/>   
	       
		 <!-- 扫描org.djj包下面的java文件，有Spring的相关注解的类，则把这些类注册为Spring的bean -->
	    <context:component-scan base-package="org.djj.hrm"/>
	    
		<!-- 使用PropertyOverrideConfigurer后处理器加载数据源参数 -->
		<context:property-override location="classpath:db.properties"/>

		<!-- 配置c3p0数据源 -->
		<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"/>
	
		<!-- 配置SqlSessionFactory，org.mybatis.spring.SqlSessionFactoryBean是Mybatis社区开发用于整合Spring的bean -->
		<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"
		    p:dataSource-ref="dataSource"/>
	
		<!-- JDBC事务管理器 -->
		<bean id="transactionManager" 
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
			 p:dataSource-ref="dataSource"/>
	
		<!-- 启用支持annotation注解方式事务管理 -->
		<tx:annotation-driven transaction-manager="transactionManager"/>
	
	</beans>

配置数据库连接池：db.properties

	dataSource.driverClass=com.mysql.jdbc.Driver
	dataSource.jdbcUrl=jdbc:mysql://127.0.0.1:3306/hrm_db
	dataSource.user=root
	dataSource.password=root
	dataSource.maxPoolSize=20
	dataSource.maxIdleTime = 1000
	dataSource.minPoolSize=6
	dataSource.initialPoolSize=5

整合spring mvc：springmvc-config.xml:

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:mvc="http://www.springframework.org/schema/mvc"
	    xmlns:context="http://www.springframework.org/schema/context"
	    xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd     
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.2.xsd">
		
	    <!-- 自动扫描该包，SpringMVC会将包下用了@controller注解的类注册为Spring的controller -->
	    <context:component-scan base-package="org.djj.hrm.controller"/>
	    <!-- 设置默认配置方案 -->
	    <mvc:annotation-driven/>
	    <!-- 使用默认的Servlet来响应静态文件 -->
	    <mvc:default-servlet-handler/>
	    
	    <!-- 定义Spring MVC的拦截器 -->
	    <mvc:interceptors>
	    	<mvc:interceptor>
	    		<!-- 拦截所有请求 -->
	    		<mvc:mapping path="/*"/>
	    		<!-- 自定义判断用户权限的拦截类 -->  
	    	 	<bean class="org.djj.hrm.interceptor.AuthorizedInterceptor"/>
	    	</mvc:interceptor>
	    </mvc:interceptors>
	    
	    
	    <!-- 视图解析器  -->
	     <bean id="viewResolver"
		  class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
		<!-- 前缀 -->
		<property name="prefix">
		    <value>/WEB-INF/jsp/</value>
		</property>
		<!-- 后缀 -->
		<property name="suffix">
		    <value>.jsp</value>
		</property>
	    </bean>
	    
	     <bean id="multipartResolver"  
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">  
			<!-- 上传文件大小上限，单位为字节（10MB） -->
		<property name="maxUploadSize">  
		    <value>10485760</value>  
		</property>  
		<!-- 请求的编码格式，必须和jSP的pageEncoding属性一致，以便正确读取表单的内容，默认为ISO-8859-1 -->
		<property name="defaultEncoding">
			<value>UTF-8</value>
		</property>
	    </bean>
	    
	</beans>

修改web.xml;

	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
		xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
		xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
		http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" 
		id="WebApp_ID" version="3.1">
	
		<!-- 配置spring核心监听器，默认会以 /WEB-INF/applicationContext.xml作为配置文件 -->
		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>
		<!-- contextConfigLocation参数用来指定Spring的配置文件 -->
		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/applicationContext*.xml</param-value>
		</context-param>
	
		<!-- 定义Spring MVC的前端控制器 -->
	  <servlet>
	    <servlet-name>springmvc</servlet-name>
	    <servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	    </servlet-class>
	    <init-param>
	      <param-name>contextConfigLocation</param-name>
	      <param-value>/WEB-INF/springmvc-config.xml</param-value>
	    </init-param>
	    <load-on-startup>1</load-on-startup>
	  </servlet>
	  
	  <!-- 让Spring MVC的前端控制器拦截所有请求 -->
	  <servlet-mapping>
	    <servlet-name>springmvc</servlet-name>
	    <url-pattern>/</url-pattern>
	  </servlet-mapping>
	  
	  <!-- 编码过滤器 -->
	  <filter>
			<filter-name>characterEncodingFilter</filter-name>
			<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
			<init-param>
				<param-name>encoding</param-name>
				<param-value>UTF-8</param-value>
			</init-param>
	 </filter>
		<filter-mapping>
			<filter-name>characterEncodingFilter</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
	
		<!-- jsp的配置 -->
	  <jsp-config>
	    <jsp-property-group>
	    	 <!-- 配置拦截所有的jsp页面  -->
	      <url-pattern>*.jsp</url-pattern>
	       <!-- 可以使用el表达式  -->
	      <el-ignored>false</el-ignored>
	      <!-- 不能在页面使用java脚本 -->
	      <scripting-invalid>true</scripting-invalid>
	      <!-- 给所有的jsp页面导入要依赖的库，tablib.jsp就是一个全局的标签库文件  -->
	      <include-prelude>/WEB-INF/jsp/taglib.jsp</include-prelude>
	    </jsp-property-group>
	  </jsp-config>
	  
	  <error-page>
	    <error-code>404</error-code>
	    <location>/404.html</location>
	  </error-page>
	  
	  <welcome-file-list>
	    <welcome-file>/index.jsp</welcome-file>
	  </welcome-file-list>
	  
	</web-app>

到这里，我们的ssm框架就完成了。


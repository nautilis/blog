---
layout: post
title: "Intellij整合spring跟mybatis，以及mapper开发方式的小坑"
categories: 教程
tags: Java
commemts: true

---
   最近在学spring跟mybatis，感觉每个框架都是博大精深，確实是需要花不少时间去研究。总觉得一边开发一边摸索着学是最好不过的了，所以先把兩个框架整合起來。
## Intellij

一开始打算用Eclipse,不过安装了sts插件之后它打开就慢的不要不要的还时不时卡死给你看,看来Eclipse真的是有点没落了，难怪大家现在一言不和都不說話并扔你一个Eclipse。相反，Intellij各种组件完备,很容易开发javaEE，快捷键加上ideaVim写起代码来也是相当带感。
## 整合目标

1.让spring ioc容器管理mybatis的sqlsessionfactory   <br>
2.让spring管理mapper

## 項目結构

在intellij新建一个webapp等待項目构建完成。新增一个java文件夾右键mark directory as sources root.最終結构如下

{% img[] /images/s.JPG %}

## 相关的jar包
通过maven下载相关的jar包xml如下<br/>

_pom.xml_

{% codeblock lang:xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>GoodGoodCode</groupId>
  <artifactId>spring_mybatis2</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>spring_mybatis2 Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>

    <!-- 导入mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.2.7</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.3.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.6.11</version>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.11</version>
    </dependency>
    <!-- c3p0数据库连接池 -->
    <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.1.2</version>
    </dependency>
    <!-- mysql connector -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.38</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>3.2.1.RELEASE</version>
    </dependency>

    <!-- spring 跟 mybatis 的整合包-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.2.3</version>
    </dependency>



  </dependencies>
  <build>
    <finalName>spring_mybatis2</finalName>
  </build>
</project>

{% endcodeblock %}

<!--more -->
## Spring跟mybatis配置

在resources下建spring包跟mybatis包分别放置各自的配置文件。

_Application context. xml_

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 加载外部数据库配置 -->
    <context:property-placeholder location="classpath:db.properties"></context:property-placeholder>

    <!-- 配置datasource-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="user" value="${jdbc.user}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="driverClass" value="${jdbc.driverclass}"></property>
        <property name="initialPoolSize" value="${jdbc.initPoolSize}"></property>
        <property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
    </bean>

    <!-- 配置SelsessionFactory,让IOC容器管理 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 配置mybatis 的配置文件-->
        <property name="configLocation" value="mybatis/sqlMapConfig.xml"/>
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 使用dao开发时，配置userDao bean -->
    <bean id="userDao" class="dao.UserDaoImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
    </bean>

    <!-- 使用mapper 方式开发时扫描 mapper包 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="mapper"/>
        <!-- 配置sqlSessionFactory 时不能用ref 要用 sqlSessionFactoryBeanName -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

</beans>

{% endcodeblock %}

_sqlMapconfig.xml_

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- Globally enables or disables any caches configured in any mapper under this configuration -->
        <setting name="cacheEnabled" value="true"/>
        <!-- Sets the number of seconds the driver will wait for a response from the database -->
        <setting name="defaultStatementTimeout" value="3000"/>
        <!-- Enables automatic mapping from classic database column names A_COLUMN to camel case classic Java property names aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>

    <!-- 配置别名 -->
    <typeAliases>
        <package name="po"/>
    </typeAliases>

    <!-- dao方式开发时加载mapper -->
    <mappers>
        <mapper resource="sqlmap/userMapper.xml"/>
    </mappers>

    <!-- Continue going here -->

</configuration>
{% endcodeblock %}

_db. properties配置數据库信息_

{% codeblock lang:java %}
jdbc.user = root
jdbc.password = bevan123
jdbc.driverclass = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql:///mybatis
jdbc.initPoolSize = 5
jdbc.maxPoolSize = 10
{% endcodeblock %}

--------

现在两个框架基本上整合到一起了.

## 兩種开发方式

**Dao方式开发：**

没什么好説的，写dao跟dao实现類(継承SqlSessionDaoSupport),然后在Ioc容器中配置下就可以用了。

**Mapper方式开发:**

不用实现接口，不过要尊循规范:

- xx\_mapper. xml中的namespace要指向对应的xx\_mapper.java
- xx_mapper中的接口方法的名字对应xx mapper. xml中的id，输入参数类型对應parameter type,输出类型对应result type
- xx\_mapper.xml跟xx\_mapper.java在同一目录下

## 小坑:
 在intellij中你会发现你已經遵循所有规范后还是报错: _org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)_ <br/>
 这是因为Maven默认只打包resources下的配置文件，而我们Mapper开发将xxMapper.xml放在java下。所以要手动配置Maven打包java下的非*.java文件。

 {% codeblock lang:xml %}
<build>
...
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <excludes>
            <exclude>**/.svn/*</exclude>
        </excludes>
    </resource>
    <resource>
        <directory>src/main/java</directory>
        <excludes>
            <exclude>**/.svn/*</exclude>
        </excludes>
        <includes>
            <include>**/*.xml</include>
        </includes>
    </resource>
</resources>
        ...
</build>
 {% endcodeblock %}


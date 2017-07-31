---
title: 使用intellij idea+maven打造最简单的spring应用
date: 2017-05-12 17:26:27
categories:
tags:
---

网上搜一下idea+spring关键字出来的都是Spring MVC项目，我只是想学习Spring，根本不需要什么web项目啊。

废话不多说，直接上流程

1. 左上角File-New-Project 
2. 选择Maven，勾选create from archetype，选择org.apache.maven.archetypes:maven-archetype-quickstart，Next输入GroupId "com.stefler" ArtifactId "spring-start"，Next，Finish.
3. 稍等一段时间，建好工程。
4. 打开pom文件，在dependencies中添加
```
<!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.2.5.RELEASE</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.2.5.RELEASE</version>
        </dependency>
```
这是spring的两个核心依赖，稍等一段时间idea会auto import所有的依赖。
5. 在src/main目录下新建文件夹resources，右击resources -> Mark Directory as -> Resources root。
6. 在刚才新建的resources目录下新建文件applicationContext.xml，用来放置spring的配置信息。
7. 在src/main/java/com/stefler目录下新建类InfoCollect，代码如下

```java
package com.stefler;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Created by Qinjiu on 5/12/2017.
 */
public class InfoCollect {
    private String name;
    private int age;
    private String address;
    private String passWord;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        InfoCollect infoCollect = (InfoCollect) context.getBean("infoCollect");
        int i = 1;
    }

}
```

9. 在applicationContext.xml中添加bean配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    <bean id="infoCollect" class="com.stefler.InfoCollect">
        <property name="name" value="stefler"/>
        <property name="age" value="111"/>
        <property name="address" value="hangzhou"/>
        <property name="passWord" value="22}"/>
    </bean>
</beans>

```

10. 直接运行刚刚InfoCollect中的main函数即可。

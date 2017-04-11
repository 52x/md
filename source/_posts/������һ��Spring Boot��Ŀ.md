title: 开发第一个Spring Boot项目
date: 2016-08-04 15:48:44
permalink: Spring-Boot-1
tags:
- Spring Boot
categories:
- Spring Boot

---


> 让我们用Spring Boot结合Maven开发一个简单的“Hello World“Java Web应用。

### 搭建Maven项目，创建pom.xml文件

创建一个普通的Maven项目，如下图所示：
![](/images/spring-boot-one/1.png)
修改pom.xml文件内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>

   <groupId>com.example</groupId>
   <artifactId>demo</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <packaging>jar</packaging>

   <name>demo</name>
   <description>Demo project for Spring Boot</description>

   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.4.0.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
   </parent>

   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <java.version>1.8</java.version>
   </properties>

   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
   </dependencies>

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>
</project>
```

pom.xml文件主要为：

```xml
<parent>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-parent</artifactId>
   <version>1.4.0.RELEASE</version>
   <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 编写代码

要启动Spring Boot 项目，我们要创建一个包含main方法的启动类，代码如下：
这里demo项目例子是一个Java Web 项目，所以一些Spring MVC的代码也写在一起了。
```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {

   @RequestMapping("/")
   public String index(){
      return "Hello World!";
   }

   public static void main(String[] args) {
      SpringApplication.run(DemoApplication.class, args);
   }
}
```
### 启动程序
 
可以直接运行main方法启动项目，也可以通过Spring Boot的maven插件启动： `mvn spring-boot:run` 
项目启动效果如下：
![](/images/spring-boot-one/2.png)

打来浏览器输入  `localhost:8080` 即可看到下面的输出：
>Hello World!


Spring Boot项目的其中一个好处就是可以打包成Jar包直接运行，下面我们来创建并运行SpringBoot项目的Jar包。

### 打包程序
首先在pom.xml文件中添加  `spring-boot-maven-plugin` ，代码如下：
```xml
<build>
   <plugins>
      <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
   </plugins>
</build>
```

然后执行maven打包命令 `mvn package`即可，打包完即可在target文件夹下看到项目的Jar包。

![](/images/spring-boot-one/3.png)

在命令行输入java 启动Jar包命令`java -jar`即可启动项目，	`ctrl-c`即可关闭项目。

>到此，第一个Spring Boot 项目开发完成！=_=














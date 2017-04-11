title: Spring Boot特点之SpringApplication
date: 2016-08-09 17:20:44
permalink: Spring-Boot-2
tags:
- Spring Boot
categories:
- Spring Boot

---


在上一篇文章[开发第一个Spring Boot项目](http://www.dongxiaoxia.xyz/2016/08/04/Spring-Boot-1/ "开发第一个Spring Boot项目")我们创建了第一个Spring Boot 项目，主要的代码为：
```java
@SpringBootApplication
public class DemoApplication {
   public static void main(String[] args) {
      SpringApplication.run(DemoApplication.class, args);
   }
}
```
在`main`方法中引用了`SpringApplication`这个类，`SpringApplication`类通过运行`main`方法提供了一个方便的方法来启动Spring Boot 应用。在许多情况下直接在`main`方法里面调用`SpringApplication`的静态方法`SpringApplication.run`即可。

### 自定义Banner

项目启动后可以看到一个奇葩的玩意：
```
.   ____          _            __ _ _
  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
  =========|_|==============|___/=/_/_/_/
  :: Spring Boot ::        (v1.4.0.RELEASE)
```

这是什么鬼=_=!!  表示看不懂。
这个是Spring Boot 项目启动时默认的Banner，既然看不懂，那我们来自定义一个自己的Banner吧。
先上效果图：
![](/images/spring-boot-two/1.png)
嗯，感觉不怎么样好像=_= ，怎么做到的呢，很简单，只要在类路径下添加  `banner.txt `文件即可。
Banner文件不但可以是 `banner.txt`   ，还可以是` banner.gif`, `banner.jpg` 或者`banner.png`  等图片，同样只要放在类路径下即可，效果如下面图片所示。
![](/images/spring-boot-two/2.png)
![](/images/spring-boot-two/3.png)
本项目直接放在`src\main\resources`下。
![](/images/spring-boot-two/4.png)
可以通过在配置文件`application.yml`下 修改`spring.main.banner-mode`属性决定 是否要在控制台输出(`console`)Banner或记录在log日志下(`log`)，或者不显示Banner图案(`off`)。
```
spring:
  main:
    banner-mode: "off"
```

### 自定义SpringApplication
如果SpringApplication默认的配置不满足要求，我们可以创建一个本地的SpringApplication实例在定制自己想要的配置。举个栗子，想要关闭banner我们可以向下面那样，不过先要注释掉application配置文件中相关配置信息。
```java
SpringApplication app = new SpringApplication(DemoApplication.class);
app.setBannerMode(Banner.Mode.OFF);
app.run(args);
```
可以使用编码的方式配置SpringApplication,当然我们也可以用`application.properties`或`application.yml`配置文件来配置。有关SpringApplication的配置选项可以查看SpringApplication [Javadoc](http://docs.spring.io/spring-boot/docs/1.4.0.RELEASE/api/org/springframework/boot/SpringApplication.html) 。

### 流畅的SpringApplication 构建API
除了使用上面的方式构建SpringApplication，还可以使用`SpringApplicationBuilder`来构建。举个栗子：
```java
new SpringApplicationBuilder()
      .bannerMode(Banner.Mode.OFF)
      .sources(Parent.class)
      .child(DemoApplication.class)
      .run(args);
```
### Application事件与监听器
除了使用Spring框架的事件之外，SpringApplication也会发送一些额外的事件。
有些事件是在`ApplicationContext`生成之前就自动触发了，对于这些事件，不能作为`@Bean`注册监听器，可以通过 `SpringApplication.addListeners(…​)` or `SpringApplicationBuilder.listeners(…​) `方法。
如果不管应用是否已经创建都想要那些监听器被自动注册，可以在项目下添加一个`META-INF / spring.factories`文件，使用 `org.springframework.context.ApplicationListener`关键字配置。如：
```
org.springframework.context.ApplicationListener=com.example.project.MyListener
```
Application事件按下列顺序发送，你的应用程序运行：
	1.  `ApplicationStartedEvent `：在运行开始前发出，除了监听器和初始化的注册之外的任何处理之前。
	2.  `ApplicationEnvironmentPreparedEvent` ： 当在上下文中使用的 Environment 是已知的，但是在创建上下文之前
	3. `ApplicationPreparedEvent` : 启动刷新之前，bean定义已经加载之后。
	4.  `ApplicationReadyEvent` ： 刷新之后，表明该应用程序已准备好服务请求的任何相关的回调已被处理 之后。
	5.  `ApplicationFailedEvent `：如果启动时有异常。

在Spring Boot内部是使用事件来处理各种任务，但通常通常我们不需要使用这些事件。

### Web环境
一个SpringApplication将尝试创建正确类型的`ApplicationContext`。默认会根据是否在开发一个web 应用使用` AnnotationConfigApplicationContext `或`AnnotationConfigEmbeddedWebApplicationContext `。决定使用web环境的算法其实是十分简单的（基于几个类的存在），想要覆盖这个默认机制可以使用 `setWebEnvironment(boolean webEnvironment)`。
另外，也可以采取通过调用` setApplicationContextClass(…​)`所使用的`ApplicationContext`的类型来完全控制。通常在JUnit测试中使用` SpringApplication`时会调用 `setWebEnvironment(false)`。

### 访问应用参数
如果你想要访问通过 `  SpringApplication.run(…​)`得到的应用参数，可以注入一个 `org.springframework.boot.ApplicationArguments` bean。 `ApplicationArguments`接口提供了访问未加工的 String[]参数以及解析后的选项和非选项参数。
```java
package com.example;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * Created by dongxiaoxia on 2016/8/9.
 */
@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args){
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }
}
```
测试一下：

![](/images/spring-boot-two/5.png)
![](/images/spring-boot-two/6.png)
![](/images/spring-boot-two/7.png)


### 使用ApplicationRunner或CommandLineRunner
如果需要在springapplication启动之后运行一些特定的代码，可以实现 `ApplicationRunner`或 ` CommandLineRunner `接口。 两个接口以相同的方式工作，并提供了一​​个单一的 `run `方法，该方法将被调用 `SpringApplication.run(…​)`完成之前。
CommandLineRunner接口提供访问用应用程序参数作为的一个简单的字符串数组，而ApplicationRunner使用上面所讨论的ApplicationArguments接口。
```java
package com.example;

import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

/**
 * Created by dongxiaoxia on 2016/8/9.
 */
@Component
public class ApplicationRunnerTest implements CommandLineRunner {
    @Override
    public void run(String... agrs) throws Exception {
        //DO something
        System.out.println("+++++++++++++++++++++++++++++++++++++++++++++++++");
        for (String arg:agrs){
            System.out.println(arg);
        }
        System.out.println("+++++++++++++++++++++++++++++++++++++++++++++++++");
    }
}
```

如果定义了几个必须在一个特定的顺序调用 的`CommandLineRunner `或 `ApplicationRunner `bean ,你还可以实现` org.springframework.core.Ordered`接口或使用 `org.springframework.core.annotation.Order`注释。

### Application退出
每一个 `SpringApplication`   都会在JVM注册一个关机钩子以确保 `ApplicationContext   `在退出时优雅的关闭。所有标准的Spring生命周期回调（ 如 `DisposableBean `接口，或 `DisposableBean `注释）都能被使用。
此外，如果想要在应用结束时返回一个具体的退出代码，可以实现 `org.springframework.boot.ExitCodeGenerator`接口。

### 管理功能
通过指定` spring.application.admin.enabled`属性，可以使行政相关的特点 。这暴露了在` MBeanServer`平台上的 [SpringApplicationAdminMXBean](https://github.com/spring-projects/spring-boot/tree/v1.4.0.RELEASE/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java)。可以使用此功能来远程管理Spring Boot程序。这对任何服务包装器实现也是有用的 。

+ <small>如果你想知道，应用程序正在运行在哪个HTTP端口 ，可以通过local.server.port获取。<small>
+ <small>启用此功能时，作为MBean暴露的关机方法应用要注意。<small>


> 本章完结撒花 V_V

































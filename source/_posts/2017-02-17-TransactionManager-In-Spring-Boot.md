---
title: Spring Boot使用什么TransactionManager
date: 2017-02-17 14:21:06
updated: 2017-02-17 14:21:06
tags: [spring,spring boot,hibernate,java]
categories: [Spring,Java]
keywords: [spring boot,spring,hibernate,TransactionManager]
description:
---

# 序

碰到一个问题，昨天到今天，花了一天的时间，才发现就错了一个小地方，特记录之

# 起因

事情起因是这样的，给写的系统，添加一个记录Activity的功能，系统使用的Spring Boot1.4.3框架，SpringJPA、Hibernate的持久层，引入Hibernate Search做全文检索。还有Druid的数据源。

开始比较顺利，写了一个Entity类，定义好字段，自动生成表，在一个内容模块中验证了，增加、更新都可以顺利的添加记录。验证成功，顺手开始添加其他的，添加到登录的信息记录时，出问题了。无报错，但是数据库也没有写入！

# 乱投医

这就稀奇了，无写入的sql语句，开Debug，设断点，没定位具体问题！这还是第一次碰到这种情况，完全没思路了，于是开始乱求医的过程

1. 考虑是不是因为在扩展的SimpleUrlAuthenticationSuccessHandler中，系统调用时没有autowire到相应的函数（大雾。。。），于是不再调用一个现成写log的helper类，手动autowire了相应的service。手动写了对象，写入，还是一样。

2. 因为现象是只有在正常调用的Request中才能写入，怀疑是OpenSessionInViewFilter导致的，事务关闭了。禁掉了OpenSessionInViewFilter，然后测试，照样失败

3. 猜测是延迟写入库了，在ServiceImpl中，吧save方法，改为了saveAndFlush方法，重启来，还是不行，不过有点进步了，有报错了。**终于看到错误信息了，泪奔！。**第一次因为看到错误信息而高兴，是不是有点犯贱。。。

   ```
   org.springframework.dao.InvalidDataAccessApiUsageException: no transaction is in progress; nested exception is javax.persistence.TransactionRequiredException: no transaction is in progress
   	at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:413)
   	at org.springframework.orm.hibernate5.HibernateExceptionTranslator.translateExceptionIfPossible(HibernateExceptionTranslator.java:55)
   ```

4. 看到了这个开始根据这个提示查找问题，没有事务啊，那就加事务呗，检查service上已经有了@Transactional了，没错啊，那就加上属性，@Transactional(propagation= Propagation.REQUIRED,readOnly = false)还是不行！

5. 继续修改，在Repository中加@Transactional，不行。

6. 继续，在调用方法层加@Transactional，还不行。

7. 针乱投医了，因为之前刚刚把Springboot框架从1.3.3升级到1.4.3，是不是框架版本问题，上官网查看，nnd，居然还是被墙了。上去，看到居然已经升到了1.5.1 遂升级spring boot框架。升级后，还是不行，注：新的框架没使用的org.json居然换了实现了，害得我又改了一些东西

8. 又猜，是不是druid给挡了？查看druid，没有block的记录啊。

9. 再猜，Hibernate Search给闹的？，于是新建一个项目，直接在ApplicationMain里面，新建了一个bean，再试。还是一样。这个时候，接近崩溃了。。。

10. 参照Hibernate一一对应参数设置，没事啊。。。

11. 受不了了，手动开session，试了几个办法都不行。。

# 错误原因

再手动开session时， 注意到了使用的TransactionManager是HibernateTransactionManager，开始没有多想，感觉挺正常的。【话说当年我脑子怎么想到用这个的？】

后来，发现调用EntityManager似乎不对啊，不能开tx？才想到按说Spring JPA加Hibernate，直接用HibernateTransactionManager似乎有点不太合适的样子，查找才发现，应该用的是实现PlatformTransactionManager 接口的JtaTransactionManager。

直接注释掉HibernateTransactionManager的bean。。。，好了。。好了。。。。。。

# 总结

大概原因是当时搭建框架时候，不知道参照哪个就把HibernateTransactionManager给配置上去了，问题是数据库访问逻辑，全在调用中，居然访问数据库都正常。也怪自己，没事非觉得Spring boot的自动配置部分不如一条条写出来安稳，当时就绕了不少弯路，还以为学费已经教的差不多了，没想到今天又碰到了一个坑。

因为看到这个提示的所有网上内容中，没有哪位和我一样的，特记录下来，聊以自我提醒吧。还好过程中也学到了一点东西。 
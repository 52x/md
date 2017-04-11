title: 利用GitHub搭建个人maven仓库
date: 2016-10-18 10:56:30
permalink: github-maven-repository
tags:
- github maven 
categories:
- maven

---

## 一.deploy项目到本地

1.在pom.xml中配置distribuManage
```xml
<distributionManagement>
    <repository>
        <id>file-repository</id>
        <url>file://D:\GitHub\maven-repo</url>
        <!--SNAPSHOT 名称下用 Maven3 uniqueVersion不起作用 ，必须用Maven2-->
        <uniqueVersion>false</uniqueVersion>
    </repository>
</distributionManagement>
```
2.执行命令 `mvn deploy`
## 二.  提交项目到GitHub
```bash
git init
git add -A
git commit -m "first commit"
git remote add origin https://github.com/dongxiaoxia/maven-repo.git
git push -u origin master
```
## 三.在其他项目中引入该依赖
```xml
    <repositories>
        <repository>
            <id>dongxiaoxia-repository</id>
            <name>东小侠的仓库</name>
            <url>https://raw.githubusercontent.com/dongxiaoxia/maven-repo/master</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>xyz.dongxiaoxia.commons</groupId>
            <artifactId>utils</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
```
## 注意问题
1. 如果项目为SNAPSHOT快照版本，deploy时生成的jar包名称会带上时间戳，`<uniqueVersion>false</uniqueVersion>` 没有效果，解决方法两种，一是去掉SNAPSHOT名称后缀，二是把maven版本maven3改为maven2，并且加上`<uniqueVersion>false</uniqueVersion>` 。
<br/>
2. 其他项目引用时可能会找不到该依赖，可以看一下maven 的setting配置文件，在我的setting文件中，配置了如下一段配置：
```
     <mirror>
        <id>nexus-osc</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus osc</name>
        <url>http://repo1.maven.org/maven2/</url>
     </mirror>
```
这样配置的话就算pom.xml文件配置了正确的repository，也没有效果，因为所有的依赖都被拦截掉了。
可以配置如下:
```
    <mirror>
        <id>nexus-osc</id>
        <mirrorOf>*,!dongxiaoxia-repository</mirrorOf>
        <name>Nexus osc</name>
        <url>http://repo1.maven.org/maven2/</url>
    </mirror>
```

>三步下来，成功利用GitHub搭建个人maven仓库并且成功引用到项目中。GitHub中的maven仓库例子请见 [maven-repo](https://github.com/dongxiaoxia/maven-repo) 。

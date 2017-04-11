title: Properties配置文件读取相关java知识
date: 2016-10-14 13:58:44
permalink: java-properties
tags:
- java
categories:
- java

---
## 一.`getResourceAsStream`方法使用
  **这里的`getResourceAsStream`主要是针对`Class`与`ClassLoader`而言的 **
1. `Class.getResourceAsStream(String path) `： path 不以’/'开头时默认是从此类所在的包下取资源，以’/'开头则是从ClassPath根下获取。其只是通过path构造一个绝对路径，最终还是由ClassLoader获取资源。 
2. `Class.getClassLoader.getResourceAsStream(String path) `：默认则是从ClassPath根下获取，path不能以’/'开头，最终是由ClassLoader获取资源。 
3. `ServletContext. getResourceAsStream(String path)`：默认从WebAPP根目录下取资源，Tomcat下path是否以’/'开头无所谓，当然这和具体的容器实现有关。 

> **获取`Class`方法**：
> 1.`XX.class    `   
>  2.`this.getClass()`
>**获取`ClassLoader`方法**：
>1.`Thread.currentThread().getContextClassLoader() `
>2.`Class.getClassLoader`


**`getResourceAsStream` 用法大致有以下几种： **

第一： 要加载的文件和.class文件在同一目录下，例如：com.x.y 下有类me.class ,同时有资源文件myfile.xml 
那么，应该有如下代码： 
me.class.getResourceAsStream("myfile.xml");
第二：在me.class目录的子目录下，例如：com.x.y 下有类me.class ,同时在 com.x.y.file 目录下有资源文件myfile.xml 
那么，应该有如下代码： 
me.class.getResourceAsStream("file/myfile.xml"); 
第三：不在me.class目录下，也不在子目录下，例如：com.x.y 下有类me.class ,同时在 com.x.file 目录下有资源文件myfile.xml 
那么，应该有如下代码： 
me.class.getResourceAsStream("/com/x/file/myfile.xml"); 
总结一下，可能只是两种写法 
第一：前面有 “   / ” 
“ / ”代表了工程的根目录，例如工程名叫做myproject，“ / ”代表了myproject 
me.class.getResourceAsStream("/com/x/file/myfile.xml"); 
第二：前面没有 “   / ” 
代表当前类的目录 
me.class.getResourceAsStream("myfile.xml"); 
me.class.getResourceAsStream("file/myfile.xml"); 
缺点： 
getResourceAsStream读取的文件路径只局限与工程的源文件夹中，包括在工程src根目录下，以及类包里面任何位置，但是如果配置文件路径是在除了源文件夹之外的其他文件夹中时，该方法是用不了的。
## 二.`Properties`配置文件的读取
### 1.使用`Apache Commons Configuration`读取配置信息
第三方标准jar包，容易使用，拥有众多功能，不用重复造轮子,但是为了读取一个配置文件就要引入一个jar包时就要考虑一下了。
### 2.`Spring ResourceLoader `
Spring 能支持入参路径的很多方式，包括已 " /"、"classpath" 开头。 
   如果你想在项目中使用 Spring 提供的默认配置文件载入实现，可以这样书写你的代码。
```java
ResourceLoader resourceLoader = new DefaultResourceLoader();
Resource resource = resourceLoader.getResource("log4j.properties");
Properties props = new Properties();
 props.load(resource.getInputStream());
```
 **缺点**：依赖于Spring框架，难以通用， 如果用Spring框架应该用Spring统一管理配置文件，而不是当作一个工具类使用。

### 3.通过 `java.util.ResourceBundle` 对象 操作　
  通过该方式仅能读取配置文件而已，不能进行写操作。示例：
```java
// ResourceBundle rb = ResourceBundle.getBundle("配置文件相对工程根目录的相对路径（不含扩展名）");
ResourceBundle rb = ResourceBundle.getBundle("config");
String name = rb.getString("name");
```
java自带的类，使用非常简单，代码量还少。但是功能略微简单。

### 4.通过`Properties`对象手动写个工具类
通过Properties对象自定义封装，更能满足自己需要的要求。
```java
 /**
     * 根据Properties配置文件名称获取配置文件对象
     *
     * @param propsFileName Properties配置文件名称（从ClassPath根下获取）
     * @return Properties对象
     */
    private Properties getProperties(String propsFileName) {
        if (propsFileName == null || propsFileName.equals("")) throw new IllegalArgumentException();
        Properties properties = new Properties();
        InputStream inputStream = null;
        try {
            try {
                if (propsFileName.lastIndexOf(PROPERTY_FILE_SUFFIX) == -1) {//加入文件名后缀
                    propsFileName += PROPERTY_FILE_SUFFIX;
                }
                //写法1：
//                inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream(propsFileName);
                //写法2：
//                URL url = Thread.currentThread().getContextClassLoader().getResource(propsFileName);
//                inputStream = url.openStream();
                //写法3：
//                inputStream = PropertiesLoader.class.getClassLoader().getResourceAsStream(propsFileName);
                //写法4：
//                URL url = PropertyUtil.class.getClassLoader().getResource(propsFileName);
//                inputStream = url.openStream();
                //写法5：
                inputStream = PropertiesLoader.class.getResourceAsStream("/" + propsFileName);
                if (null != inputStream) properties.load(inputStream);
            } finally {
                if (null != inputStream) {
                    inputStream.close();
                }
            }
        } catch (IOException e) {
//            LOGGER.error("加载属性文件出错!", e);
            e.printStackTrace();
            throw new RuntimeException(e.getMessage(), e);
        }
        return properties;
    }
```
## 三.jar包读取外部和内部配置文件
### 1.获取文件的方式
把项目打成jar包发布后jar中的文件就不能通过File file=new File("文件路径")的方式来读取文件了。
错误方式：
```java
URL url = DBHelper.class.getClassLoader().getResource(configName);
URL url = DBHelper.class.getResource("/db.properties");
```
 从依赖的Jar包中读取文件, Jar包内的文件是无法用File读取的，只能用Stream的方式读取。 
正确方式：
```java
InputStream inputStraean = DBHelper.class.getClassLoader().getResourceAsStream("db.properties")
```
结论：**不能传文件路径，只能传输入流**
### 2.测试获取文件的各种情况
待测试jar包目录：
![Alt text](/images/java-properties/1.png)
测试目录引用进来的maven目录
![Alt text](/images/java-properties/2.png)
测试项目目录
![Alt text](/images/java-properties/3.png)
待测试jar包运行结果：
![Alt text](/images/java-properties/4.png)
测试项目运行结果
![Alt text](/images/java-properties/5.png)

根据测试得到的结果：
1. 项目与项目引入jar包的文件都是在classpath下面的，也就是可以通过getResourceAsStream获取，jar包与项目的获取方法没有分别，也就是说，获取jar内部与外部文件的方式是一样的。
2. 如果项目与jar包在相对于classpath下有同样的文件，则以项目的文件覆盖jar包里面的文件。
3. jar包里面没有d.properties，所有jar包里面测试运行没有找到，但是到了项目里面，在项目下创建d.properties,可是可以发现该文件的。
4. 项目可以获取项目的文件
项目可以获取jar包里面的文件
jar包可以获取项目的文件
jar包可以获取jar包里面的文件
项目与jar包同时存在该文件，则以项目的文件优先

因此，如果开发一个库或jar包，可以保留一份默认配置在jar包里面，也可以把配置文件放到客户端配置，检测时做处理，如果客户端配置该文件，则以客户端的配置为先，没有找到就用默认配置文件，或者别的处理（抛异常等。。。），slf4j配置我猜也是这样子吧=_=。

## 四.Properties配置文件读取工具类
```java
package xyz.dongxiaoxia.commons.utils;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.*;
import java.util.logging.Logger;

/**
 * Properties文件载入工具，可载入多个Properties文件，相同属性在最后载入的文件中的值将会覆盖之前的值，但以System的Property优先。
 *
 * @author dongxiaoxia
 * @create 2016-07-01 23:03
 */
public class PropertiesLoader {

    private static Logger logger = Logger.getLogger(PropertiesLoader.class.getName());

    /**
     * .properties属性文件名后缀
     */
    public static final String PROPERTY_FILE_SUFFIX = ".properties";

    private final Properties properties;

    public PropertiesLoader(String... propsFileNames) {
        properties = loadProperties(propsFileNames);
    }

    public PropertiesLoader(String propsFileName) {
        properties = getProperties(propsFileName);
    }

    public PropertiesLoader(InputStream inputStream) {
        properties = loadProperties(inputStream);
    }

    public Properties getProperties() {
        return properties;
    }

    /**
     * 取出Property，但与System的Property优先，取不到返回空字符串。
     *
     * @param key
     * @return
     */
    private String getValue(String key) {
        String systemProperty = System.getProperty(key);
        if (systemProperty != null) {
            return systemProperty;
        }
        if (properties.containsKey(key)) {
            return properties.getProperty(key);
        }
        return "";
    }

    /**
     * 取出String类型的Property，但与System的Property优先，如果都为NUll则抛出异常。
     *
     * @param key
     * @return
     */
    public String getProperty(String key) {
        String value = getValue(key);
        if (value == null) {
            throw new NoSuchElementException();
        }
        return value;
    }

    /**
     * 取出String类型的Property，但以System的Property优先，如果都为Null则返回Default值。
     *
     * @param key
     * @param defaultValue
     * @return
     */
    public String getProperty(String key, String defaultValue) {
        String value = getValue(key);
        return value != null ? value : defaultValue;
    }

    /**
     * 取出Integer类型的Property。但以System的Property优先，如果都为null或内容错误则抛异常。
     *
     * @param key
     * @return
     */
    public Integer getInteger(String key) {
        String value = getValue(key);
        if (value == null) {
            throw new NoSuchElementException();
        }
        return Integer.valueOf(value);
    }

    /**
     * 取出Integer类型的Property，但以System的Property优先。如果都为null则返回默认值，如果内容错误则抛异常。
     *
     * @param key
     * @param defaultValue
     * @return
     */
    public Integer getInteger(String key, Integer defaultValue) {
        String value = getValue(key);
        return value != null ? Integer.valueOf(value) : defaultValue;
    }

    /**
     * 取出Double类型的Property，但以System的Property优先，如果都为null或内容错误则抛异常。
     *
     * @param key
     * @return
     */
    public Double getDouble(String key) {
        String value = getValue(key);
        if (value == null) {
            throw new NoSuchElementException();
        }
        return Double.valueOf(value);
    }

    /**
     * 取出Double类型的Property，但以System的Property优先，如果都为null则返回默认值，如果内容错误则抛异常。
     *
     * @param key
     * @param defaultValue
     * @return
     */
    public Double getDouble(String key, Double defaultValue) {
        String value = getValue(key);
        return value != null ? Double.valueOf(value) : defaultValue;
    }

    /**
     * 取出Boolean类型的Property，但以System的Property优先，如果都为null则抛出异常，如果内容不是true/false则返回false。
     *
     * @param key
     * @return
     */
    public Boolean getBoolean(String key) {
        String value = getValue(key);
        if (value == null) {
            throw new NoSuchElementException();
        }
        return Boolean.valueOf(value);
    }

    /**
     * 取出Boolean类型的Property，但以System的Property优先，如果都为null则返回默认值，如果内容不为true/false则返回false。
     *
     * @param key
     * @param defaultValue
     * @return
     */
    public Boolean getBoolean(String key, boolean defaultValue) {
        String value = getValue(key);
        return value != null ? Boolean.valueOf(value) : defaultValue;
    }

    public Set<Object> getAllKey() {
        return properties.keySet();
    }

    public Collection<Object> getAllValues() {
        return properties.values();
    }

    public Map<String, Object> getAllKeyValue() {
        Map<String, Object> mapAll = new HashMap<>();
        Set<Object> keys = getAllKey();
        for (Object key1 : keys) {
            String key = key1.toString();
            mapAll.put(key, properties.get(key));
        }
        return mapAll;
    }

    /**
     * 根据Properties配置文件名称获取配置文件对象
     *
     * @param propsFileName Properties配置文件名称（从ClassPath根下获取） 可以不带扩展名
     *                      eg.根目录下有个common.properties,那么可以传“common”或者“common.properties”
     *                      根目录下有个config文件夹，里面存在common.properties,那么可以传“config/common”或者“config/common.properties”
     * @return Properties对象
     */
    private Properties getProperties(String propsFileName) {
        if (propsFileName == null || propsFileName.equals("")) throw new IllegalArgumentException();
        Properties properties = new Properties();
        InputStream inputStream = null;
        try {
            try {
                if (propsFileName.lastIndexOf(PROPERTY_FILE_SUFFIX) == -1) {//加入文件名后缀
                    propsFileName += PROPERTY_FILE_SUFFIX;
                }
                //写法1：
//                inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream(propsFileName);
                //写法2：
//                URL url = Thread.currentThread().getContextClassLoader().getResource(propsFileName);
//                inputStream = url.openStream();
                //写法3：
                inputStream = PropertiesLoader.class.getClassLoader().getResourceAsStream(propsFileName);
                //写法4：
//                URL url = PropertyUtil.class.getClassLoader().getResource(propsFileName);
//                inputStream = url.openStream();
                //写法5：
//                inputStream = PropertiesLoader.class.getResourceAsStream("/" + propsFileName);
                if (null != inputStream) properties.load(inputStream);
            } finally {
                if (null != inputStream) {
                    inputStream.close();
                }
            }
        } catch (IOException e) {
//            LOGGER.error("加载属性文件出错!", e);
            e.printStackTrace();
            throw new RuntimeException(e.getMessage(), e);
        }
        return properties;
    }

    /**
     * 载入多个文件
     *
     * @param propsFileNames
     * @return
     */
    private Properties loadProperties(String... propsFileNames) {
        Properties pros = new Properties();
        for (String propsFileName : propsFileNames) {
            InputStream inputStream = null;
            try {
                inputStream = PropertiesLoader.class.getClassLoader().getResourceAsStream(propsFileName);
                pros.load(inputStream);
            } catch (Exception e) {
                logger.info("Could not load properties from path:" + propsFileName + "," + e.getMessage());
            } finally {
                try {
                    if (inputStream != null) {
                        inputStream.close();
                    }
                } catch (IOException e) {
                    logger.info(e.getMessage());
                }
            }
        }
        return pros;
    }

    /**
     * 根据输入流载入Properties对象
     *
     * @param inputStream
     * @return
     */
    private Properties loadProperties(InputStream inputStream) {
        Properties pros = new Properties();
//        pros.load(inputStream);
        try (InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "UTF-8")) {
            pros.load(inputStreamReader);
        } catch (Exception e) {
            logger.info(e.getMessage());
        }
        return pros;
    }
}


```

更多请查看GItHub [PropertiesLoader](https://github.com/dongxiaoxia/commons/blob/master/utils/src/main/java/xyz/dongxiaoxia/commons/utils/PropertiesLoader.java)类


---
title: Eclipse注释规范模版总结
date: 2016-08-17 16:37:49
tags: Java
toc: true
categories: Java平台
---

## 具体操作

- 在eclipse中，打开Window->Preference->Java->Code Style->Code Template

- 然后展开Comments节点就是所有需设置注释的元素，参照下面注释规范对应设置即可

## 注释规范

### 文件(Files)注释标签
```java
/**
 * FileName:     	${file_name}
 * @Description: 	${todo}
 * @author:    		crane-yuan
 * @version    		V1.0
 * Createdate:      ${date} 	${time}
 * Copyright:    	Copyright(C) 2016-2017
 * Company       	CY.
 * All rights Reserved, Designed By crane-yuan

 * Modification  History:
 * Date         Author        Version        Discription
 * ---------------------------------------------------------------------------
 * ${date}     crane-yuan       1.0             1.0

 * Why & What is modified:

 */
```

<!--more-->

### 类型(Types)注释标签（类的注释）：
```java
/**
 * @ClassName:   	${type_name}
 * @Description:	${todo}
 * @author:    		crane-yuan
 * @date:			${date}		${time}
 */
```
 
### 字段(Fields)注释标签：

``` java
 /**  
  * @Fields ${field} :			${todo}
  */   
```

### 构造函数(Constructor)注释标签：
```java
/**
 * @Title:        	${enclosing_type}
 * @Description:    ${todo}
 * @param:    		${tags}
 * @throws
 */ 
```

### 方法(Methods)标签：
```java
/**
 * @Title: 			${enclosing_method}
 * @Description: 	${todo}
 * @param: 			${tags}   
 * @return: 		${return_type}   
 * @throws
 */
 ```
 
### 覆盖方法(Overriding Methods)标签:
```java
/**
 * <p>Title: ${enclosing_method}</p>
 * <p>Description: </p>
 * ${tags}
 * ${see_to_overridden}
 */ 
 ```
 
### 代表方法(Delegate Methods)标签：
```java
/**
 * ${tags}
 * ${see_to_target}
 */ 
```

### getter方法标签：
```java
/**
 * @Title:        	${enclosing_method} <BR>
 * @Description:  	${todo} <BR>
 * @return:     	${field_type} <BR>
 */ 
```

### setter方法标签：
```java
/** 
 * @Title:  		${enclosing_method} <BR> 
 * @Description: 	${todo} <BR>
 * @return: 		${field_type} <BR> 
 */  
```

---
title: React-Native 之index.ios.js解读
tags: index.ios.js
date: 2017-03-22 11:23:30
categories: react-native学习笔记
toc: true
---

撸代码之前还是要搞清楚作用比较好，不然洋洋洒洒的撸了个天昏地暗，却不知道为什么用这个姿势撸，不就尴尬了吗？要知其然，还要知其所以然，论掌握撸代码姿势的重要性！
<!--more-->

**************************************************************
React.native是facebook开源的一套基于JavaScript的开源框架，
很方便用来开发移动设备的app。
而且，方便及时更新app的UI与数据，也很方便部署。
**************************************************************

在react-native的ios项目中，界面搭建部分是一个js文件：index.ios.js。下面对这个文件进行一下解读，方便后续开发中明白各个部分的作用。

## **1、引用React**

```
import React, { Component } from 'react';
```

## **2、控件的引入**

```
import {
AppRegistry,
StyleSheet,
Text,
View,
ListView,
TouchableOpacity
} from 'react-native';

```

## **3、样式设置**

```
//设置样式
const styles = StyleSheet.create({
container: {
flex: 1,
justifyContent: 'center',
alignItems: 'center',
backgroundColor: '#F5FCFF',
},
welcome: {
fontSize: 20,
textAlign: 'center',
margin: 10,
},
instructions: {
textAlign: 'center',
color: '#333333',
marginBottom: 5,
},
});
```

作用：定义了一段应用在 “Hello World” 文本上的样式。

**React Native 使用 CSS 来定义应用界面的样式。**

## **4、创建React组件对应的类**

```
export default class HelloWorld extends Component {
render() {
return (
<View style={styles.container}>
<Text style={styles.welcome}>
Welcome to React Native!
</Text>
<Text style={styles.instructions}>
To get started, edit index.android.js
</Text>
<Text style={styles.instructions}>
Double tap R on your keyboard to reload,{'\n'}
Shake or press menu button for dev menu
</Text>
</View>
);
}
}
```

作用：描述将要创建的组件，包括各种行为和属性。

## **5、解释一下**



 - 组件渲染的方法

```
render() {
return ();
}
```
注意：只有当组件被渲染时，必须实现render接口方法，因为，只有render方法，是用于输出内容组件内容的；其他接口方法，都是可选的。

 - 标签<View>定义视图

```
<View style={styles.container}>
</View>
```
作用：设置显示区域，相当于iOS中的UIView控件（Objective-c和Swift）

 - 标签<Text>定义文本

```
<Text style={styles.welcome}>
Welcome to React Native!
</Text>
```
作用：设置并显示字符串，相当于iOS的UILabel控件（Objective-c和Swift）。

 - 定义程序入口

```
AppRegistry.registerComponent('HelloWorld', () => HelloWorld);
```

作用：用AppRegistry的registerComponent( )方法，定义了App的入口，并提供了根组件。


转自：http://blog.csdn.net/maoyingyong/article/details/46439951

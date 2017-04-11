---
title: React-Native 之ListView
tags: listView
categories: react-native学习笔记
toc: true
date: 2017-03-22 10:19:00
---

最近在摸索`react-native`，虽然苹果爸爸已经在之前封杀了`JSPatch`，我还是抱着试一试的态度先学一个疗程，毕竟，知识嘛，多学点总是好的。
<!--more-->

其实对于js我了解的不多，所以一些东西给不了相应的解释，还请见谅（ps：我的学习阶段都是从模仿开始的）。后面我会不断的学习基础知识，把相应的解释会添加上去的。见笑了！

接下来先学习一下如何创建一个`ListView`。

## **1.设置样式**

上代码：

```
//设置样式
const styles = StyleSheet.create({

//整个listView的样式设置
outerViewStyle: {
//占满窗口
flex: 1
},

//一个自定义view的样式设置
headerViewStyle: {
height: 64,
backgroundColor: 'orange',
justifyContent: 'center',
alignItems: 'center'
},

//列表row的样式设置
rowStyle: {
//设置主轴的方向
flexDirection: 'row',
//侧轴方向居中
alignItems: 'center',

padding: 10,
//单元格底部的线设置
borderBottomColor: '#e8e8e8',
borderBottomWidth: 0.5
},

//分区头部view的样式设置
sectionHeaderViewStyle: {
backgroundColor: '#e8e8e8',
justifyContent: 'center',
height: 25
}
});
```

以上就是本listView能用到的一些设置。

## **2.获取数据**

用到的数据是本地的json数据

```
//调取数据
componentDidMount(){
this.loadDataFromJson();
},

var Car = require('./Car.json');
loadDataFromJson(){
//获取json数据
var jsonData = Car.data;

//定义一些变量
var dataBlob = {},
sectionIDs = [],
rowIDs = [],
cars = [];

for (var i = 0; i < jsonData.length; i++) {
//1.把区号放入sectionIDs数组中
sectionIDs.push(i);

//2.把区中的内容放入dataBlob对象中
dataBlob[i] = jsonData[i].title;

//3.取出该组中所有的车
cars = jsonData[i].cars;
rowIDs[i] = [];

//遍历所有的车数组
for (var j = 0; j < cars.length; j++) {
//1.把行号放入rowIDs[i]中
rowIDs[i].push(j);
//2.把每一行的内容放入dataBlob对象中
dataBlob[i + ':' + j] = cars[j];
}
}

//更新状态
this.setState({
dataSource: this.state.dataSource.cloneWithRowsAndSections(dataBlob,sectionIDs,rowIDs)
});
},
```

## **3.初始化函数**

```
//初始化函数
getInitialState(){

//配置区数据
var getSectionData = (dataBlob,sectionID) => {
return dataBlob[sectionID];
};

//配置行数据
var getRowData = (dataBlob,sectionID,rowID) => {
return dataBlob[sectionID + ':' +rowID];
};

return {
dataSource : new ListView.DataSource({

getSectionData: getSectionData,//获取区中的数据
getRowData: getRowData,//获取行中的数据
rowHasChanged: (r1,r2) => r1 !== r2,
sectionHeaderHasChanged: (s1,s2) => s1 !== s2

})

}

},

render() {
return (<ListView />);
},
```


## **4.配置数据**

```
// 每一行的数据
renderRow(rowData){
return(
<TouchableOpacity activeOpacity={0.5}>
<View style={styles.rowStyle}>
<Text style={{marginLeft:5}}>{rowData.name}</Text>
</View>
</TouchableOpacity>
);
},

renderSectionHeader(sectionData,sectionID) {
return(
<View style={styles.sectionHeaderViewStyle}>
<Text style={{marginLeft:5,color:'red'}}>{sectionData}</Text>
</View>
);
}
```

## **5.界面显示**

```
render(){
return (
<View style = {styles.outerViewStyle}>
<View style={styles.headerViewStyle}>
<Text style={{color:'white',fontSize:25}}>车的品牌</Text>
</View>
<ListView
dataSource={this.state.dataSource}
renderRow={this.renderRow}
renderSectionHeader={this.renderSectionHeader}
/>
</View>
);
},
```

上一个效果图：

![这只是一个效果图](http://img.blog.csdn.net/20170322095734121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVGhyZWVfWmhhbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


由于本人也是刚刚窥探rn，所以很多地方都是不求甚解，所以很多地方没有给出相应的解释，还请见谅！这里给出源码，大家可以共同学习！

[怒戳我，得源码！](https://github.com/ZJQian/RNStudyListView/tree/master)



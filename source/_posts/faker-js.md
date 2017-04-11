title: Faker.js
date: 2015-08-24 08:05:30
tags: [javascript,node]
---

Faker.js 能够生成名字、地址、电话号码等信息的库。
<!--more-->

项目地址： https://github.com/Marak/faker.js

使用方法比较简单：
* 浏览器中

```javascript
<script src = "faker.js" type = "text/javascript"></script>
<script>
  var randomName = faker.name.findName(); // Caitlyn Kerluke
  var randomEmail = faker.internet.email(); // Rusty@arne.info
  var randomCard = faker.helpers.createCard(); // random contact card containing many properties
</script>
```

```javascript
ar faker = require('faker');

var randomName = faker.name.findName(); // Rowan Nikolaus
var randomEmail = faker.internet.email(); // Kassandra.Haley@erich.biz
var randomCard = faker.helpers.createCard(); // random contact card containing many properties
```

Faker的API中包含了大量的方法，并且，有针对本地化的语言生成可以使用`faker.locale = 'zh_CN'`。

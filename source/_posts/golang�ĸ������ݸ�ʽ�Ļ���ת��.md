title: golang的各种数据格式的互相转换
date: 2015-12-27 12:50:30
tags: [golang]
---

#### int to string
```js
import (   
    "strconv"  
) 

int i = 10
 str1 := strconv.Itoa(i)  
```

#### struct to json
```js
import (
    "encoding/json"
)

type Server struct {
    ServerName string
    ServerIP   string
}

b, err := json.Marshal(s)
if err != nil {
   fmt.Println("json err:", err)
}
fmt.Println(string(b))
```
未完待续
title: SpringMVC Controller接收List< Map>或List< Object>
date: 2015-12-23 20:58:44
permalink: spring00001
tags:
- Spring
categories:
- Spring

---
google了一个下午，暂时找到两种解决办法，虽然这两种方法都不是很方便，暂时先这么用着吧！
## 1.全部参数封装为一个对象
### controller类
```java
/**
 * 批量新增用户
 *
 * 不能直接传List  < Map> 数组对象，只能用两个法子传参数，一个是把数组对象在前端变为String传到后台再解析，另外一个就是把所有参数封装成一个对象，前端JSON.stringify(params)后变为字符串传到后台，后台会自动转换为对象
 *
 * @param batchCreateVO 封装好的对象
 * @param request
 * @param session
 * @return
 */
@RequestMapping(value = "/batchCreate", produces = MediaTypes.JSON_UTF_8)
@ResponseBody
protected String batchCreate(
        @RequestBody BatchCreateVO batchCreateVO,
        HttpServletRequest request, HttpSession session) {
        //dosomething
	}
```

### 封装的对象
```java
package com.miracle.mby.account.vo;

import java.util.List;
import java.util.Map;

/**
 * 移动端批量创建用户封装对象
 * <p/>
 * ticket       身份校验ticket
 * mac          身份校验mac
 * refDeptId    企业顶级部门
 * usersString  用户对象数组字符串
 *
 * @author dongxiaoxia
 * @create 2015-12-22 14:33
 */
public class BatchCreateVO {
    private String ticket;
    private String mac;
    private String refDeptId;
    private List<Map> users;

    public String getTicket() {
        return ticket;
    }

    public void setTicket(String ticket) {
        this.ticket = ticket;
    }

    public String getMac() {
        return mac;
    }

    public void setMac(String mac) {
        this.mac = mac;
    }

    public String getRefDeptId() {
        return refDeptId;
    }

    public void setRefDeptId(String refDeptId) {
        this.refDeptId = refDeptId;
    }

    public List<Map> getUsers() {
        return users;
    }

    public void setUsers(List<Map> users) {
        this.users = users;
    }
}
```

### 页面请求
```javascript
var users = [{"userName": "测试用户1","sex":0,"mobile":"15432343213","email":"123@123.com","phone":"123213","departmentIds":[]}];
var params ={
    "ticket":"test",
    "mac":"test",
    "refDeptId":"fa0823254d6011e58434ac220bcd3c54",
    "users":users
};
$.ajax({
    type: 'POST',
    url: 'http://localhost:8081/api/mobile/user/batchCreate',
    data: JSON.stringify(params),
    dataType: 'json', contentType: "application/json; charset=utf-8"
});
```


## 2.单独转换需要的List
单独把List< Map>或List< Object> 参数转换为json字符串，然后传到后台之后再把字符串转换会对象






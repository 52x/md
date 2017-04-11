title: 532movie iOS v1.0 开发总结
date: 2015-12-06 14:53:02
categories: iOS
tags: [swift]
---
# 综述
原计划开学初上线的App，由于种种原因导致12月初才submitted。最近两个月完成了iOS部分的开发、设计、测试、运维，在本文中对开发过程进行记录。
<!-- more -->

# 一、开发背景
532Movie上线以来，一直面临高峰期卡顿问题，尤其今年上半年。2015年春季，后台大概是2台服务器在运转，平日每天吞吐量5TB，高峰期间每小时吞吐1TB。1TB/H什么概念呢？换算下单位就是`291MB/s`！对于普通的硬盘来说，100MB+/s的I/O读写不卡顿就是奇迹，不崩溃就是侥幸。相比于千兆带宽的网络，此时I/O读写显然成为短板。浪哥接连使用HLS协议将原有mp4视频切片，只有使用fastDFS分布式存储文件，最后用Varnish做缓存系统。一翻折腾过后，集群里面大概有5台服务器，独立出1台服务器处理web请求缓存。高峰期大部分用户都在看热门影片，这些影片的片段在cache中的命中率异常高，既保护了磁盘也提高了I/O速度。至此后台改造基本完成，那下一步怎么办？？机房的千兆带宽绝不是我们有能力改变的，当网络变成瓶颈的时候，如何还能进一步优化？
后台优化完了，那就写个APP优化前端吧
1.移动端屏幕小，可以在APP中播放低码率视频，节省高峰期带宽。
2.平峰时期开放离线下载功能，为高峰期分流。
3.客户端提供了P2P的想象空间，万不得已时，可以将热门影片P2P，不占用机房出口带宽。


# 二、 数据分析
网站中内嵌了Google Analytics，开发前简单观测了统计数据如下：
![2015年11月 Pageviews](/images/532movie/v1/pageview201511.png)
![2015年11月 Users](/images/532movie/v1/users201511.png)
`注:网站仅限校内网访问，因此用户访问网站时如果没有连接外网，Google无法统计`
上图可以看出访问量成明显的周期性波动，周末高峰期每日接近10万的pageviews，1万多的独立用户。师大每届只有2000多的本科生，全校本科生只有1万多人，这意味着大部分同学每天都会访问此网站。那么其中有多少iOS用户呢？
![2015年11月 操作系统比例](/images/532movie/v1/operatingsystem201511.png)
大概有12.6%的用户使用iOS系统浏览网站，这还是很令我意外的。由于web页面没有对mobile做任何定制，所以网站在手机浏览器中用户体验并不能令人满意，但即使这样每天扔有1k+的同学使用iOS浏览网站。这坚定了我们的信息，哪怕只为了这1000多名同学，我们也要将iOS客户端写好。如果客户端的体验足够友好，恐怕会吸引更多的同学转战移动平台。
下面我关心的就是，如何选择适配的最低系统。我难以估量市场上各个版本系统所占比例，但是大学生群体相对更追求时尚，系统升级率会高于市场平均。果然Google Analytics的数据支持了我的想法：
![iOS系统版本比例](/images/532movie/v1/systemversion.png)
只有3%左右的设备是iOS 8.0以下的系统，那么放弃iOS 8.0以下的设备不会对用户造成太大影响，同时可以增加选择新技术的自由度。

通过简略的分析，可以得出
1.几乎全校学生都在浏览532的网站，给这样的网站开发APP是很有必要的
2.首先开发用户数更高的iOS设备
3.最低版本支持到iOS 8.0


# 三、 技术架构
## 客户端
### 语言选择
2013年前后曾经用iOS写过两个单机的小APP，当时用的OC语言，这次开发前在纠结使用OC还是用Swift。看了一天Swift语法就不再纠结了，Swift用的很舒服，容易上手，但是想精通还是很难得。大概用两周时间看了斯坦福的cs193P iOS8公开课，其间穿插看《The Swift Programming Language (Swift 2.1)》这本书。
### 系统设计
自己摸索着市场上的APP，设计了下拉刷新、加载更多、缓存、播放器手势等功能，自己搭建后台确定API接口。
### 编码开发
iOS代码大概写了三五天，UI基本使用Story Board。三年前做iOS开发的时候好像只有一种比例的屏幕，随着屏幕的多样化，苹果出了Auto Layout布局方式，核心的Constraint源于一篇线性约束的论文，去简单读了下。好在532movie的布局不复杂，Auto Layout基本可以搞定。好久没写OC，但是大致可以看懂OC语法，因此程序中也混编了OC代码。绝大部分时间都在调试UICollectionView的性能问题，最终做到了丝滑体验。后期在做缓存设计，APP的服务器是校网内网IP，所以APP经常会无法访问后台，不想让空白的页面呈现在用户面前，选择了用key-value的方式存储JSON数据。
### 开源库
主要使用的开源库如下：
#### [Alamofire](https://github.com/Alamofire/Alamofire)
AFNetworking的作者发布的Swift版本，如作者所说，这个库的确非常Elegant。阅读了Alamofire的源码，有种想重敲一边的冲动，作者用很短的代码优雅的实现了HTTP Networking。
#### [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON)
SwiftyJSON makes it easy to deal with JSON data in Swift.可以和Alamofire配合使用。
#### [Kingfisher](https://github.com/onevcat/Kingfisher)
onevcat大大写的类似于SDWebImage的图片库，操作都是异步进行的，不会阻塞UI进程。多级缓存设计，可以将图片缓存在内存和硬盘中。
#### [YTKKeyValueStore](https://github.com/yuantiku/YTKKeyValueStore)
### 测试上线
找同学帮忙拿到BNU内网环境测试，12.4日拿到开发者账号，晚上将App提交，变为waiting for review状态。

## 四、服务器端
后台开发是由浪哥一人完成的，有个执行力很好地后台开发人员真的很爽。每次提出需求没有多久，浪哥就会顺利搞定。
### 本地环境 nginx+Virtualenv+Flask
设计后台API接口的时候，自己搭建了简易后台。为了使用RESTful，选用了Flask-RESTful，足够达到测试的目的。

### 部署环境 nginx+ThinkPHP
532Movie Web版本就是这种环境，开发API的时候，直接在原有基础上加了一个模块。后台最大的问题是，532Movie的服务器在师大内网，外网无法访问。之前还可以通过IPv6或者VPN访问，但是学校11月刚刚把532Movie加入VPN的黑名单，台湾的大学竟然不支持IPv6。苹果公司审核App的时候，也需要App后台支持外网访问。最后经过简单的讨论，我们使用如下解决方案：
1.我在App中不请求IP地址，请求域名，浪哥将大陆教育网DNS指向BNU校内服务器，其余DNS指向美国的DigitalOcean服务器。
2.在DigitalOcean(DO)中搭建虚拟的532Movie后台，数据库和图片资源只有几百兆，可以直接迁移到DO服务器。但是影片资源不可能迁移到DO上面。之前写过一篇文章，介绍在DO上面搭建shadowsocks，利用DO的IPv6，实现在中国高校免费上网。现在我们将它倒转过来，在DO设置nginx反向代理，将来自IPv4的请求，通过IPv6访问校内服务器获取资源。
最后的结果是，全网可以访问532Movie后台，当在BNU外网访问影片资源时，经过DigitalOcean的IPv6跳转。

### Trello
[Trello](https://trello.com/)是一个scrum任务管理工具，免费，快捷，很适合个人和团队使用。我在上面记录todo和doing list，通过card管理任务的优先级。
目前的todo list是：
-评论回复模块
-收藏、订阅影片
-播放记录
-推送消息
-离线下载
-弹幕

# 五、感受
完成App的“功能”开发很容易，但是对“细节”和“性能”的优化是一条很长的路，玩了那么多年算法，终于可以找个地方实践一下了。






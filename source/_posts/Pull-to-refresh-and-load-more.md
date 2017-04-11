title: iOS下拉刷新与加载更多
date: 2015-12-09 17:12:28
categories: iOS
tags: [swift]
---

![瀑布流展示图片](/images/UICollectionView/pull to refresh.png)

<!-- more -->

# 下拉刷新
[DGElasticPullToRefresh](https://github.com/gontovnik/DGElasticPullToRefresh)是一款swift书写的弹性下拉刷新控件。
![DGElasticPullToRefresh预览](/images/UICollectionView/DGElasticPullToRefreshPreview.gif)
UIScrollView的子类都可以使用该控件。

不考虑本地缓存问题的话，下拉刷新功能非常简单，每次下拉时向后台API发送请求。返回结果存入Model中，reload即可。
api接口为 ../api/v1/videoinfos/count/50
在UICollectionView中有一个小trick，与UITableView不同的是，collectionView中的cells不足一屏幕时，是不能拖拽滚动条的。(UITableView哪怕没有cells也可以拖拽)。以竖直方向scroll为例`collectionView?.alwaysBounceVertical = true`可以使collectionView在任何情况下都能竖直拖拽。


# 加载更多
Web开发中常用的分页方式，在移动端并不适用。如果用户加载更多的同时，有新数据插入，那么就会导致客户端的内容出现重复。这种判重可以在客户端进行，也可以在后台进行，我们选择在后台进行数据运算。客户端只需把列表中最后一条信息的addtime传给后台，服务器按时间排序后，将这个addtime之前的count条信息返回。
为和刷新接口同步，在../api/v1/videoinfos中加入addtime参数，addtime=0或者addtime为空时返回最新的count条信息，否则返回addtime小于给定值的前count条信息。（其中addtime是每部影片的UNIX时间戳，保证数据库中addtime唯一性）

在处理好拼接逻辑后，另一个问题是何时发起加载更多请求。我们希望在用户不知不觉中，将数据加载完毕。重载scrollViewDidScroll，用户滚动到90%的时候，自动发起加载更多请求。
```swift
    override func scrollViewDidScroll(scrollView: UIScrollView) {
        if scrollView.contentOffset.y + view.frame.size.height > scrollView.contentSize.height * 0.9 { // Loads more videos
            if let lastadd = videos.last?.addtime {
                if lastadd != lastAddTime && lastAddTime != ""{
                    populateVideoInfos()
                }
            }
        }
    }
```
这种设计虽然方便了用户，但是要解决两个问题。
`问题1，判断是否已没有更多数据`
如果不进行判断，用户停留在scroll底部时，将会不断向服务器发送请求，造成网络压力过大。解决方法很简单，可以在返回数据中加入一项lastaddtime，即所有影片中addtime最小值。在发送请求前判断本地列表中是否已经包含lastaddtime。

`问题2，在同一位置可能会触发多次加载更多请求`
滚动到底部时，会多次发送加载更多请求，他们的返回结果都是重复的。借鉴[Alamofire Tutorial Part 1: Getting Started](http://www.raywenderlich.com/85080/beginning-alamofire-tutorial)文中的方法，设置populatingVideoInfos bool变量，开始loadmore操作时，将populatingVideoInfos锁死，结束loadmore时在恢复populatingVideoInfos。对populatingVideoInfos的操作不要放在UI线程，要放在网络请求线程。

# 缓存设计
之前开发过一版直接套用UIWebView的App，当没有网络时，应用完全空白，非常影响用户体验。所以无论从性能角度，还是用户体验角度，都要对数据进行缓存。
## 图片缓存
图片资源非常占用网络资源，Kingfisher可以直接将网络图片持久化存储在本地。532Movie约有2000部影片，每部影片的封面50kb左右，将全部图片资源存储起来也只需要100MB空间，因此将全部图片缓存是可行的。

## 数据缓存
Kingfisher缓存的原理是以图片URL为key值进行hash。如果不将影片信息缓存，就无法获得图片url。影片信息如何缓存呢？
可以使用Core Data存储，但是会比较heavy。唐巧师兄开源了[YTKKeyValueStore](https://github.com/yuantiku/YTKKeyValueStore)。首先Nosql也可以实现复杂的业务逻辑，其次移动端存储的数据并不会很大。YTKKeyValueStore是对FMDB进行简单封装，底层是sqlite3。网上的资料显示小于20KB的文件，sqlite效率要高于Core Data。我们可以将服务器返回的json数据直接作为value存储起来。50条影片信息的json文件，只占12kb，存入sqlite毫无压力。
在iPhone6真机测试YTKKeyValueStore的效率，100KB的json读写100次，耗时2.5s，写入2s，读取0.5s。测试时使用的是系统自带的sqlite库，到官网下载最新的sqlite速度会更快。

### 增量更新
目前影片最多的一个分类下面，大致有900条影片，将他们全部缓存大概需要40ms，相比于网络请求，这样的时间是可以接受的。

目前的缓存逻辑是，用户refresh时，首先会从硬盘读取缓存信息，同时发起网络请求。将网络请求回的50条信息，增量加入本地列表。如果网络返回数据与本地列表没有交集，则用返回数据更新本地缓存，只有当用户不频繁访问App时才存在这种情况。

以上的更新策略是假设客户端与服务器的影片时序完全相同，由于两边都采用addtime排序，只要影片的addtime不变，那么两边的顺序就不会有问题。对于影片来说，只会有一个addtime，但是对于正在更新的电视剧，addtime会是最新剧集的加入时间。因此电视剧的的排序会随着新剧集的加入而改变，之前的更新策略就不适用了。

解决方案如下：
对于网络返回的每条数据，我们查找他在本地列表中的addtime，不相同则将列表中的记录删除。本地列表最多有500条电视剧信息，网络返回50条信息。如果使用数组存储，按addtime二分查找时间复杂度是O(log500*50)=O(500)。删除一条记录的最坏复杂度O(500)，由于存在重复的剧集不会太多，因此这种删除操作次数也不会很多，所以这种方案的时间效率可以接受。


---
title: moya + RxSwift 进行网络请求
tags: moya + RxSwift
categories: swift学习笔记
date: 2017-03-13 17:59:00
toc: true
---

如在OC中使用AFNetworking一般,Swift我们用Alamofire来做网络库.而Moya在Alamofire的基础上又封装了一层:
<!--more-->

## **1.关于moya**

![moya](http://img.blog.csdn.net/20170313171002333?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVGhyZWVfWmhhbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

官方说`moya`有以下特性-_-:

- 编译时检查正确的API端点访问.
- 使你定义不同端点枚举值对应相应的用途更加明晰.
- 提高测试地位从而使单元测试更加容易.

## **2.开始**

 ### **1.创建枚举API**

就像这样:

```
enum APIManager {
case getNewsLatest//获取最新消息
case getStartImage// 启动界面图像获取
case getVersion(String)//软件版本查询
case getThemes//主题日报列表查看
case getNewsDetail(Int)//获取新闻详情
}

```

 ### **2.实现`TargetType`协议**

就像这样:

```
extension APIManager: TargetType {

/// The target's base `URL`.
var baseURL: URL {

return URL.init(string: "http://news-at.zhihu.com/api/")!
}

/// The path to be appended to `baseURL` to form the full `URL`.
var path: String {

switch self {

case .getNewsLatest:
return "4/news/latest"

case .getStartImage://start-image 后为图像分辨率，接受任意的 number*number 格式， number 为任意非负整数，返回值均相同。
return "4/start-image/1080*1776"

case .getVersion(let version)://URL 最后部分的数字代表所安装『知乎日报』的版本
return "4/version/ios/" + version

case .getThemes:
return "4/themes"

case .getNewsDetail(let id):
return "4/news/\(id)"

}


}

/// The HTTP method used in the request.
var method: Moya.Method {

return .get
}

/// The parameters to be incoded in the request.
var parameters: [String: Any]? {

return nil
}

/// The method used for parameter encoding.
var parameterEncoding: ParameterEncoding {

return URLEncoding.default
}

/// Provides stub data for use in testing.
var sampleData: Data {

return "".data(using: String.Encoding.utf8)!
}

/// The type of HTTP task to be performed.
var task: Task {

return .request
}

/// Whether or not to perform Alamofire validation. Defaults to `false`.
var validate: Bool {

return false
}

}

```

在这里,可以设置请求的参数,例如url......method......para等.

 ### **3.使用**

`Moya`的使用非常简单，通过`TargetType`协议定义好每个`target`之后，就可以直接使用`Moya`开始发送网络请求了。就像这样:

```
let provider = MoyaProvider<APIManager>()
provider.request(.getNewsLatest) { result in
// do something with result
}
```

## **3.配合RxSwift**

`Moya`本身已经是一个使用起来非常方便，能够写出非常简洁优雅的代码的网络封装库，但是让`Moya`变得更加强大的原因之一还因为它对于`Functional Reactive Programming`的扩展，具体说就是对于`RxSwift`和`ReactiveCocoa`的扩展，通过与这两个库的结合，能让`Moya`变得更加强大。我选择`RxSwift`的原因有两个，一个是`RxSwift`的库相对来说比较轻量级，语法更新相对来说比较少，我之前用过`ReactiveCocoa`，一些大版本的更新需求重写很多代码，第二个更重要的原因是因为`RxSwift`背后有整个`ReactiveX`的支持，里面包括`Java`，`JS`，`.Net`, `Swift`，`Scala`，它们内部都用了`ReactiveX`的逻辑思想，这意味着你一旦学会了其中的一个，以后可以很快的上手`ReactiveX`中的其他语言。

`Moya`提供了非常方面的`RxSwift`扩展：

```
let provider = RxMoyaProvider<APIManager>()
provider.request(.getNewsLatest)
.filterSuccessfulStatusCodes()
.mapJSON()
.subscribe(onNext: { (json) in
//do something with posts
print(json)
})
.addDisposableTo(disposeBag)
```

**解释一下:**

- `RxMoyaProvider`是`MoyaProvider`的子类，是对`RxSwift`的扩展

- `filterSuccessfulStatusCodes()`是`Moya`为`RxSwift`提供的扩展方法，顾名思义，可以得到成功地网络请求，忽略其他的

- `mapJSON() `也是`Moya RxSwift`的扩展方法，可以把返回的数据解析成 `JSON` 格式

- `subscribe` 是一个`RxSwift`的方法，对经过一层一层处理的 `Observable` 订阅一个 `onNext` 的 `observer`，一旦得到 `JSON` 格式的数据，就会经行相应的处理

- `addDisposableTo(disposeBag)` 是 `RxSwift` 的一个自动内存处理机制，跟` ARC`有点类似，会自动清理不需要的对象。


## **4.配合HandyJSON**

在实际应用过程中网络请求往往紧密连接着数据层（`Model`），具体地说，在我们的这个例子中，一般我们需要建立一个类用来统一管理数据，然后把得到的 `JSON` 数据映射到数据层（`Model`）。

```
struct MenuModel: HandyJSON {
var others: [ThemeModel]?

}

struct ThemeModel: HandyJSON {

var color: String?
var thumbnail: String?
var id: Int?
var description: String?
var name: String?
}

```

然后创建ViewModel类,创建具体请求方法:

```
class MenuViewModel {

private let provider = RxMoyaProvider<APIManager>()
var dispose = DisposeBag()

func getThemes(completed: @escaping (_ menuModel: MenuModel) -> ()){

provider
.request(.getThemes)
.mapModel(MenuModel.self)
.subscribe(onNext: { (model) in

completed(model)
}, onError: { (error) in

}, onCompleted: nil, onDisposed: nil).addDisposableTo(dispose)

}

}

```

**这里解释一下:**
我这里是将请求的数据通过闭包传了出去,当然也可以不那么做.个人喜好问题..


这里是为 `RxSwift` 中的 `ObservableType`和 `Response`写一个简单的扩展方法 `mapModel`，利用我们写好的`Model` 类，一步就把` JSON `数据映射成 `model`。
```
extension ObservableType where E == Response {
public func mapModel<T: HandyJSON>(_ type: T.Type) -> Observable<T> {
return flatMap { response -> Observable<T> in
return Observable.just(response.mapModel(T.self))
}
}
}

extension Response {
func mapModel<T: HandyJSON>(_ type: T.Type) -> T {
let jsonString = String.init(data: data, encoding: .utf8)
return JSONDeserializer<T>.deserializeFrom(json: jsonString)!
}
}

```


## **5.配合ObjectMapper**

毕竟将json数据转换成model的库那么多 ....,所以......,用哪个很随意.....这里再介绍一下`ObjectMapper`

**1.创建model类**


```
class DetailModel: Mappable {

var body = String()
var image_source: String?
var title = String()
var image: String?
var share_url = String()
var js = String()
var recommenders = [[String: String]]()
var ga_prefix = String()
var section: DetailSectionModel?
var type = Int()
var id = Int()
var css = [String]()





func mapping(map: Map) {

body <- map["body"]
image_source <- map["image_source"]
title <- map["title"]
image <- map["image"]
share_url <- map["share_url"]
js <- map["js"]
recommenders <- map["recommenders"]
ga_prefix <- map["ga_prefix"]
section <- map["section"]
type <- map["type"]
id <- map["id"]
css <- map["css"]
}
required init?(map: Map) {

}
}

```

使用 `ObjectMapper` ，需要让自己的 `Model` 类使用 `Mappable` 协议，这个协议包括两个方法：

```
required init?(map: Map) {}

func mapping(map: Map) {}
```
在 `mapping` 方法中，用 `<-` 操作符来处理和映射你的 `JSON `数据。




数据类建立好之后，我们还需要为 `RxSwift` 中的 `Observable` 写一个简单的扩展方法 `mapObject`，利用我们写好的`model` 类，一步就把`JSON` 数据映射成一个个 `model`。

```
extension Observable {
func mapObject<T: Mappable>(type: T.Type) -> Observable<T> {
return self.map { response in
//if response is a dictionary, then use ObjectMapper to map the dictionary
//if not throw an error
guard let dict = response as? [String: Any] else {
throw RxSwiftMoyaError.ParseJSONError
}

return Mapper<T>().map(JSON: dict)!
}
}

func mapArray<T: Mappable>(type: T.Type) -> Observable<[T]> {
return self.map { response in
//if response is an array of dictionaries, then use ObjectMapper to map the dictionary
//if not, throw an error
guard let array = response as? [Any] else {
throw RxSwiftMoyaError.ParseJSONError
}

guard let dicts = array as? [[String: Any]] else {
throw RxSwiftMoyaError.ParseJSONError
}

return Mapper<T>().mapArray(JSONArray: dicts)!
}
}
}

enum RxSwiftMoyaError: String {
case ParseJSONError
case OtherError
}

extension RxSwiftMoyaError: Swift.Error { }

```

- `mapObject` 方法处理单个对象，`mapArray` 方法处理对象数组。

- 如果传进来的数据 `response` 是一个 `dictionary`，那么就利用 `ObjectMapper` 的 `map`方法映射这些数据，这个方法会调用你之前在 `mapping `方法里面定义的逻辑。

- 如果 `response` 不是一个 `dictionary`， 那么就抛出一个错误。

- 在底部自定义了简单的`Error`，继承了` Swift` 的 `Error `类，在实际应用过程中可以根据需要提供自己想要的 `Error`。



然后运行请求方法:

```
class DetailViewModel {

private let provider = RxMoyaProvider<APIManager>()

func getNewsDetail(id: Int) -> Observable<DetailModel> {

return provider
.request(.getNewsDetail(id))
.filterSuccessfulStatusCodes()
.mapJSON()
.mapObject(type: DetailModel.self)

}
}

```

有没有感觉很爽呢!------------[源码地址,共同学习!](https://github.com/ZJQian/ZhiHuNewsSwift)


**有不对之处,,,,还望各路大神不吝指正!**

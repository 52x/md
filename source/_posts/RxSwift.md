---
title: RxSwift的简单使用
date: 2017-03-13 08:52:06
categories: swift学习笔记
tags: RxSwift
toc: true
---

`RxSwift`为`ReactiveX`（`Reactive Extensions`）旗下的`Swift`语言库，提供了Swift平台上进行响应式编程的解决方案。Rx的重要角色为`Observable`（被观察者）和`Observer`（观察者），`Observable`类似于`ReactiveCoco`a中的`Signal`，里面装有事件流，供`Observer`订阅。事件流<!--more-->在Rx中与`ReactiveCocoa`一样具有三类：`Next`、`Error`、`Completed`，代表着继续事件、错误事件、完成事件。我们在使用`RxSwift`进行`iOS`开发时，通常会引入另外一个库：`RxCocoa`，这个库将`UIKit`以及`Foundation`框架中许多成员，如视图(`View`)、控制事件(`Control Event`)、键值观察（`KVO`）、通知（`Notification`）等等进行与`RxSwift`接入的扩展，将Rx与iOS API无缝连接。

## **1. bindTo**

`bindTo`为`ObservableType`协议的几个重载方法（`Observable`也会实现`ObservableType`协议）。顾名思义，它会将某个东东与一个可观察者进行绑定，也就是说，当这个可观察者的事件流中有事件“流过”（有事件元素发送），被绑定的这个东东就会被刺激到，进而进行相关的操作。

在这里，有一个用的比较多的是重载方法为`bindTo<O : ObserverType where O.E == E>(observer: O) -> Disposable`，这个方法有一个参数，从方法泛型的声明中可以得知，参数的类型为一个观察者类型，且这个观察者能够接受到的事件流元素的类型要跟被观察者的一样(O.E == E)。这个方法意图就是将一个被观察者与一个指定的观察者进行绑定，被观察者事件流中发出的所有事件元素都会让观察者接收。
在MVVM架构模式中，此方法主要用于视图（View）层跟视图模型（ViewModel）层或视图层跟视图层的绑定，这里举个栗子：

```
textField.rx_text
.bindTo(label.rx_text)
.addDisposableTo(disposeBag)

```

`其中，UITextField的rx_text属性为ControlProperty类型，实现了ControlPropertyType，所以不仅是观察者类型，还是被观察者类型，UILabel中的rx_text只是单纯的观察者类型。`

`bindTo`的另外一个用得比较多的重载方法为：`bindTo(variable: RxSwift.Variable<Self.E>) -> Disposable`，这个方法将一个被观察者与一个`Variable`（变量）绑定在一起，这个变量的元素类型跟被观察者的事件元素类型一致。此方法作用就是把从被观察者事件流中发射出的事件元素存入变量中，在这里不做演示。
关于bindTo的其他重载方法在这里就不完全阐述了，剩下的主要是用于对函数的绑定。


## **2.UIBindingObserver**

现在介绍的这个东东就跟上面说的被观察者类型的`bindTo`方法密切相关了。
`UIBindingObserver`，名字就告诉了我们它是一个观察者，用于对UI的绑定，我这里通过一个例子来讲解它：

```
//  MARK: - 绑定方法
func binding() {
textField.rx_text
.bindTo(label.rx_sayHelloObserver)
.addDisposableTo(disposeBag)
}
//  MARK: - 视图控件扩展
private extension UILabel {
var rx_sayHelloObserver: AnyObserver<String> {
return UIBindingObserver(UIElement: self, binding: { (label, string) in
label.text = "Hello \(string)"
}).asObserver()
}
}
```

上面的代码中，我在视图控制器`ViewController`所在的`Swift`文件中创建了一个私有的`UILabel`扩展，并在扩展中定义了一个只读计算属性，属性的类型为`AnyObserver<String>`，为一个事件元素是`String`的观察者类型。当获取这个属性值的时候，就返回了与特定`UIBindingObserver`关联的观察者。
现在我们来看一下`UIBindingObserver`的构造方法：

```
init(UIElement: UIElementType, binding: (UIElementType, Value) -> Void)
```

方法的第一个参数就是传入一个要被绑定的视图的实例，由于现在是在`UILabel`的扩展中，所以这里我传入了`self`，代表`UILabel`自己；构造方法的第二个参数为一个无返回值的闭包类型，闭包的参数其一就是被绑定了的视图，其二就是由绑定的被观察者中所发射出来的事件元素。通过这个闭包，我们能够将视图中的某些属性根据相应的事件元素而进行改变，如例子中`label.text = "Hello \(string)"`。当我们执行例子中的`binding`函数进行绑定后，`TextField`中的字符串每经过修改，`Label`中的文字总会实时更新，并在字符串前面加上`Hello`。

在`RxCocoa`框架中，某些地方也用到了`UIBindingObserver`，如`UILable`中的`rx_text`：

```
public var rx_text: AnyObserver<String> {
return UIBindingObserver(UIElement: self) { label, text in
label.text = text
}.asObserver()
}
```

## **3.Driver**

`Driver`从名字上可以理解为`驱动`（我自己会亲切地把它叫做"老司机"），在功能上它类似被观察者（`Observable`），而它本身也可以与被观察者相互转换（`Observable: asDriver, Driver: asObservable`），它驱动着一个观察者，当它的事件流中有事件涌出时，被它驱动着的观察者就能进行相应的操作。一般我们会将一个`Observable`被观察者转换成`Driver`后再进行驱动操作：

我们沿用上面例子中的`UILabel`私有扩展，并修改下`binding`方法：

```
func binding() {
textField.rx_text
.asDriver()
.drive(label.rx_sayHelloObserver)
.addDisposableTo(disposeBag)
}
```

可见，`Driver`的`drive`方法与`Observable`的方法`bindTo`用法非常相似，事实上，它们的作用也是一样，说白了就是被观察者与观察者的绑定。那为什么`RxSwift`的作者又搞出`Driver`这么个东西来呢？
其实，比较与Observable，Driver有以下的特性：

1. 它不会发射出错误(Error)事件
2. 对它的观察订阅是发生在主线程(UI线程)的
3. 自带shareReplayLatestWhileConnected


下面就围绕着这三个特性一一研究下：



-  当你将一个`Observable`转换成`Driver`时，用到的`asDriver`方法有下面几个重载：

```
asDriver(onErrorJustReturn onErrorJustReturn: Self.E)

asDriver(onErrorDriveWith onErrorDriveWith: RxCocoa.Driver<Self.E>)

asDriver(onErrorRecover onErrorRecover: (error: ErrorType) -> RxCocoa.Driver<Self.E>)
```

从这三个重载方法中可看出，当我们要将有可能会发出错误事件的`Observable`转换成`Driver`时，必须要先将所有可能发出的错误事件滤除掉，从而使得`Driver`不可能会发射出错误的事件。

- 在`Observable`中假如你要进行限流，你要用到方法`throttle(dueTime: RxSwift.RxTimeInterval, scheduler: SchedulerType)`，方法的第一个参数是两个事件之间的间隔时间，第二个参数是一个线程的有关类，如我要在主线程中，我可以传入`MainScheduler.instance`。而在`Driver`中我们要限流，调用的是`throttle(dueTime: RxSwift.RxTimeInterval)`，只配置事件的间隔时间，而它默认会在主线程中进行。


- 一般我们在对`Observable`进行`map`操作后，我们会在后面加上`shareReplay(1)`或`shareReplayLatestWhileConnected`，以防止以后被观察者被多次订阅观察后，`map`中的语句会多次调用：

```
let rx_textChange = textField.rx_text
.map { return "Good \($0)" }
.shareReplay(1)
rx_textChange
.subscribeNext { print("1 -- \($0)") }
.addDisposableTo(disposeBag)
rx_textChange
.subscribeNext { print("2 -- \($0)") }
.addDisposableTo(disposeBag)
```


在`Driver`中，框架已经默认帮我们加上了`shareReplayLatestWhileConnected`，所以我们也没必要再加上`"replay"`相关的语句了。

从这些特性可以看出，`Driver`是一个专门针对于`UI`的特定可观察者类。并不是说对`UI`进行相应绑定操作不能使用纯粹的`Observable`，但是，`Driver`已经帮我们省去了好多的操作，让我们对`UI`的绑定更加的高效便捷。所以，对`UI`视图的绑定操作，我们首选“老司机”`Driver`。


## **4.DisposeBag**

当一个`Observable`（被观察者）被观察订阅后，就会产生一个`Disposable`实例，通过这个实例，我们就能进行资源的释放了。
对于`RxSwift中`资源的释放，也就是解除绑定、释放空间，有两种方法，分别是显式释放以及隐式释放：

- **显式释放** 可以让我们在代码中直接调用释放方法进行资源的释放，如下面的实例：

```
let dispose = textField.rx_text
.bindTo(label.rx_sayHelloObserver)
dispose.dispose()
```

这个例子只是为了更明朗地说明显式释放方法而已，实际上并不会这样写。

- **隐式释放** 则通过DisposeBag来进行，它类似于Objective-C ARC中的自动释放池机制，当我们创建了某个实例后，会被添加到所在线程的自动释放池中，而自动释放池会在一个RunLoop周期后进行池子的释放与重建；DisposeBag对于RxSwift就像自动释放池一样，我们把资源添加到DisposeBag中，让资源随着DisposeBag一起释放。如下实例：

```
let disposeBag = DisposeBag()
func binding() {
textField.rx_text
.bindTo(label.rx_sayHelloObserver)
.addDisposableTo(self.disposeBag)
}
```
方法`addDisposableTo`会对`DisposeBag`进行弱引用，所以这个`DisposeBag`要被实例引用着，一般可作为实例的成员变量，当实例被销毁了，成员`DisposeBag`会跟着销毁，从而使得`RxSwift`在此实例上绑定的资源得到释放。

对于`UITableViewCell`跟`UICollectionViewCell`来说，`DisposeBag`也能让cell在重用前释放掉之前被绑定的资源：

```
class TanTableViewCell: UITableViewCell {
var disposeBag: DisposeBag?
var viewModel: TanCellViewModel? {
didSet {
let disposeBag = DisposeBag()
viewModel?.title
.drive(self.textLabel!.rx_text)
.addDisposableTo(disposeBag)
self.disposeBag = disposeBag
}
}

override func prepareForReuse() {
super.prepareForReuse()
self.disposeBag = nil
}
}
```

## **5.DataSource**

这里主要讲解的是`RxCocoa`框架中带有的对于`UITableView`以及`UICollectionView`数据源的解决方案，在`GitHub`中也有一个开源小库`RxDataSource`，在这里我就不再研究了，有兴趣的朋友可以去看看：[GitHub RxDataSource](https://github.com/RxSwiftCommunity/RxDataSources)。
我这里用一个例子来展示下`RxCocoa`中的简单`UITableView`数据源：

```
class TanViewController: UIViewController {

var disposeBag = DisposeBag()

let data = [TanCellViewModel(title: "One"), TanCellViewModel(title: "Two"), TanCellViewModel(title: "Three")]

override func viewDidLoad() {
super.viewDidLoad()
self.view.addSubview(self.tableView)
self.tableView.frame = self.view.bounds

self.binging()
}

private func binging() {
Observable.just(self.data)
.asDriver(onErrorJustReturn: [])
.drive(self.tableView.rx_itemsWithCellIdentifier(TanTableViewCell.CELL_IDENTIFIER, cellType: TanTableViewCell.self)) { (_, viewModel, cell) in
cell.viewModel = viewModel
}
.addDisposableTo(self.disposeBag)
}

//  MARK: - Lazy
private var tableView: UITableView = {
let tableView = UITableView(frame: CGRectZero, style: .Plain)
tableView.registerClass(TanTableViewCell.self, forCellReuseIdentifier: TanTableViewCell.CELL_IDENTIFIER)
return tableView
}()

}
```

如上，我们能够将数据封装在`Observable`中，然后把`Observable`绑定到`UITableView`中，通过`UITableView`的方法`rx_itemsWithCellIdentifier`，我们就能够进行数据跟Cell的一一对应配置。
到此，`UITableView`的数据源就设置好了。`UICollectionView`的数据源设置跟`UITableView`差不多，在这里就不再作例子了。

## **6.项目实战**

下面就是重头戏了，我将通过折腾出一个小项目来演示`RxSwift`的使用，包括基础以及进阶的内容，首先来设定下这个项目：
说简单点，就是做一个登录界面（万能Demo），输入用户号码跟密码，点击登录按钮，即可登录获取数据。
说复杂点，我们要完成下面的要求：

1. 用户号码输入框要判断用户输入的是否全是数字，若格式不正确，提示用户格式错误。
2. 号码输入框输入的数字最少要有11位，密码输入框输入的字符串长度最少要有6位。
3. 要满足上面的两条要求，登录按钮才可以点击。
4. 登录按钮点击后进行登录，界面显示正在转动的等待视图，当接收到后台数据时，等待视图消失。
5. 解析后台返回的数据，并把数据呈现到界面中。

在这个项目中，我还是使用熟悉的MVVM架构模式。在开干之前我首先要说几点：

- ` RxSwift`中的`ViewModel`是没有什么明确的状态的，它的输出由输入决定，可以这么说，我们要使用`RxSwift`将`ViewModel`中的外界输入（UI触发、外界事件）转换成输出，再由这些输出去驱动UI界面，并且，`ViewModel`做的是转换，我们不能够在其中对某个`Observable`进行订阅操作，所以，在`ViewModel`中我们是看不到`addDisposableTo`的。

- 我对比了一下由`ReactiveCocoa`与`RxSwift`实现的`ViewModel`，发现使用`ReactiveCocoa`实现的`ViewModel`中会有比较多的明确状态变量，比如说现在实现的是登录的界面，在`ReactiveCocoa`的`ViewModel`中我们会看到有`"userName"`、`"passWord`"等等之类的状态变量，它是由`ReactiveCocoa`将其与UI视图属性相绑定的：`RAC(self.viewModel, userName) = userNameTextField.rac_textSignal`;，而在`RxSwift`实现的`ViewModel`，就不会看到这些状态变量了，有的是驱动外界UI的输出`Driver`，个人认为`RxSwift`实现`ViewModel`的宗旨是将外界视图的输入经过转变产生输出，在让输出去驱动回UI视图，所以我在构建`ViewModel`类的时候，会在它的构造方法中开设一个接收输入的参数，其次就在后面的控制器绑定中将`ViewModel`的输出进行订阅，驱动视图层。

- 这个项目我使用的第三方库有`RxSwift`、`RxCocoa`、`Moya`、`Argo`、`Curry`，前面两个在上面有说到；`Moya`是一款`Swift`语言的网络请求框架，它是另一款网络请求框架`Alamofire`的再度封装，它有基于`RxSwift`的扩展，能与`RxSwift`无缝对接；`Argo`是一款小巧的`JSON`解析库，函数柯里化(Currying)库`Curry`配合着它一起使用，而且，`Argo`的解析语法非常新颖奇特，用着感觉非常过瘾！


### **<1>.界面**

在`Storyboard`中布局好登录界面，分别有用户电话号码的输入框、用户密码输入框、等待视图（菊花）、提示视图（用于提醒输入的错误，以及登录的状态）、登录按钮：

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1484579-61761f1db78fe2b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### **<2>.Entity 实体**

下面进行实体类(Entity)的构建：

```
import UIKit
import RxSwift
import RxCocoa
import Argo
import Moya
import Curry

//  MARK: - User
struct User {
let name: String
let userToken: String
}

extension User: Decodable {
static func decode(json: JSON) -> Decoded<User> {
return curry(self.init)
<^> json <| "name"
<*> json <| "user_token"
}
}

//  MARK: - ResponseResult
enum ResponseResult {
case succeed(user: User)
case faild(message: String)

var user: User? {
switch self {
case let .succeed(user):
return user
case .faild:
return nil
}
}
}

extension ResponseResult: Decodable {
init(statusCode: Int, message: String, user: User?) {
if statusCode == 200 && user != nil {
self = .succeed(user: user!)
}else{
self = .faild(message: message)
}
}

static func decode(json: JSON) -> Decoded<ResponseResult> {
return curry(self.init)
<^> json <| "status_code"
<*> json <| "message"
<*> json <|? "user"
}
}

//  MARK: - ValidateResult
enum ValidateResult {
case succeed
case faild(message: String)
case empty
}


infix operator ^-^ {}
func ^-^ (lhs: ValidateResult, rhs: ValidateResult) -> Bool {
switch (lhs, rhs) {
case  (.succeed, .succeed):
return true
default:
return false
}
}

//  MARK: - RequestTarget
enum RequestTarget {
case login(telNum: String, password: String)
}

extension RequestTarget: TargetType {
var baseURL: NSURL {
return NSURL(string: "")!
}

var path: String {
return "/login"
}

var method: Moya.Method {
return .POST
}

var parameters: [String: AnyObject]? {
switch self {
case let .login(telNum, password):
return ["tel_num": telNum, "password": password]
default:
()
}
}

var sampleData: NSData {
let jsonString = "{\"status_code\":200, \"message\":\"登录成功\", \"user\":{\"name\":\"Tangent\",\"user_token\":\"abcdefg123456\"}}"
return jsonString.dataUsingEncoding(NSUTF8StringEncoding)!
}
}
```

- **User** 用户类，登录成功后，后台会返回用户的个人信息，包括用户名称以及用户的登录令牌。

- **ResponseResult**
网络请求返回类，枚举类型，成功的话它的关联值是一个用户类型，失败的话它就会有信息字符串关联。它的构造中靠的是状态码来完成，若后台返回的状态码为`200`，表示登录成功，返回用户，若为其他，表明登录失败，并返回错误信息。这里的`decode`方法为`Argo`解析所需实现的。

- **ValidateResult** 验证类，如验证电话号码是否格式正确，号码或密码的长度是否达到要求等等，失败的时候会有错误信息相关联。

- **RequestTarget** 请求目标，为`Moya`框架定制的网络请求类。

### **<3>.ViewModelServer 服务**

```
import UIKit
import RxCocoa
import RxSwift
import Moya
import Argo

//  MARK: - ValidateServer
class ValidateServer {
static let instance = ValidateServer()

class func shareInstance() -> ValidateServer {
return self.instance
}

let minTelNumCount = 11
let minPasswordCount = 6

func validateTelNum(telNum: String) -> ValidateResult {
guard let _ = Int(telNum) else { return .faild(message: "号码格式错误") }
return telNum.characters.count >= self.minTelNumCount ? .succeed : .faild(message: "号码长度不足")
}

func validatePassword(password: String) -> ValidateResult {
return password.characters.count >= self.minPasswordCount ? .succeed : .faild(message: "密码长度不足")
}
}

//  MARK: - NetworkServer
class NetworkServer {
static let instance = NetworkServer()

class func shareInstace() -> NetworkServer {
return self.instance
}

//  Lazy
private lazy var provider: RxMoyaProvider = {
return RxMoyaProvider<RequestTarget>(stubClosure: MoyaProvider.ImmediatelyStub)
}()

func loginWork(telNum: String, password: String) -> Driver<ResponseResult> {
return self.provider.request(.login(telNum: telNum, password: password))
.mapJSON()
.map { jsonObject -> ResponseResult in
let decodeResult: Decoded<ResponseResult> = decode(jsonObject)
return try decodeResult.dematerialize()
}
.asDriver(onErrorJustReturn: .faild(message: "网络或数据解析错误！"))
}
}
```

在这里有两个服务类，第一个为验证服务类，用于验证用户号码格式以及号码或密码的长度是否达到要求，第二个为网络请求类，用于向后台请求登录，这里要注意的是，`RxMoyaProvider`一定要被类引用，否则若把它设置为局部变量，请求就不能完成。在构建`RxMoyaProvider`的时候，我在构造方法中传入了`MoyaProvider.ImmediatelyStub`这个`stubClosure`参数，为的是测试，这样子系统就不会请求网络，而是直接通过获取`Target`的`sampleData`属性。

### **<4>.ViewModel 视图模型**

```
import UIKit
import RxSwift
import RxCocoa

class ViewModel {
//  MARK: - Output
let juhuaShow: Driver<Bool>
let loginEnable: Driver<Bool>
let tipString: Driver<String>

init(input: (telNum: Driver<String>, password: Driver<String>, loginTap: Driver<Void>),
dependency: (validateServer: ValidateServer, networkServer: NetworkServer)) {

let telNumValidate = input.telNum
.distinctUntilChanged()
.map { return dependency.validateServer.validateTelNum($0) }

let passwordValidate = input.password
.distinctUntilChanged()
.map { return dependency.validateServer.validatePassword($0) }

let validateString = [telNumValidate, passwordValidate]
.combineLatest { result -> String in
var validateString = ""
if case let .faild(message) = result[0] {
validateString = "\(message)"
}
if case let .faild(message) = result[1] {
validateString = "\(validateString) \(message)"
}
return validateString
}

let telNumAndPassWord = Driver.combineLatest(input.telNum, input.password) { ($0, $1) }

let loginString = input.loginTap.withLatestFrom(telNumAndPassWord)
.flatMapLatest {
return dependency.networkServer.loginWork($0.0, password: $0.1)
}
.map { result -> String in
switch result {
case let .faild(message):
return "登录失败 \(message)"
case let .succeed(user):
return "登录成功，用户名:\(user.name)，标识符:\(user.userToken)"
}
}

self.loginEnable = [telNumValidate, passwordValidate]
.combineLatest { result -> Bool in
return result[0] ^-^ result[1]
}

self.juhuaShow = Driver.of(loginString.map{_ in false}, input.loginTap.map{_ in true})
.merge()

self.tipString = Driver.of(validateString, loginString)
.merge()
}
}
```

`ViewModel`相对来说比较难搞，毕竟我们要处理好每一个输入输出的关系，灵活进行转变。在这里，没有显式的状态变量，只有对外的输出以及构造时对内的输入，思想就是将输入流进行加工转变成输出流，数据在传输中能够单向传递。

### **<5>.ViewController 视图控制器**

```
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {

@IBOutlet weak var telNumTF: UITextField!
@IBOutlet weak var passWordTF: UITextField!
@IBOutlet weak var juhuaView: UIActivityIndicatorView!
@IBOutlet weak var loginBtn: UIButton!
@IBOutlet weak var tipLb: UILabel!

private var viewModel: ViewModel?
private var disposeBag = DisposeBag()

override func viewDidLoad() {
super.viewDidLoad()
self.viewModel = ViewModel(input: (
self.telNumTF.rx_text.asDriver(),
self.passWordTF.rx_text.asDriver(),
self.loginBtn.rx_tap.asDriver()),
dependency: (
ValidateServer.shareInstance(),
NetworkServer.shareInstace())
)
//  Binding
self.viewModel!.juhuaShow
.drive(self.juhuaView.rx_animating)
.addDisposableTo(self.disposeBag)

self.viewModel!.loginEnable
.drive(self.loginBtn.rx_loginEnable)
.addDisposableTo(self.disposeBag)

self.viewModel!.tipString
.drive(self.tipLb.rx_text)
.addDisposableTo(self.disposeBag)

}

}

private extension UIButton {
var rx_loginEnable: AnyObserver<Bool> {
return UIBindingObserver(UIElement: self, binding: { (button, bool) in
self.enabled = bool
if bool {
button.backgroundColor = UIColor.greenColor()
}else{
button.backgroundColor = UIColor.redColor()
}
}).asObserver()
}
}
```

在这里，我们构建好`ViewModel`，将输入以及视图模型依赖的服务传入`ViewModel`构造方法中，并在下面把`ViewModel`的输入去驱动UI视图。

[源码地址](https://github.com/TangentW/RxLoginTest)

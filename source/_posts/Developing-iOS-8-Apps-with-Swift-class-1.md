title: Developing iOS 8 Apps with Swift 学习笔记1
date: 2015-10-29 23:14:24
categories:
tags:
---
lecture 1. Logistics, iOS 8 Overview
![mvc](/images/Developing iOS 8 Apps with Swift/cs193p.jpg)
  <!-- more -->

## what's in iOS
### Cocoa Touch

  Multi-Touch    
  
  Alerts
  
  Core Motion   
  
  Web View
  
  View Hierarchy  
  
  Map Kit
  
  Localization   
  
  Image Picker
  
  Controls       
  
  Camera

### Media

  Core Audio      
  
  JPEG, PNG, TIFF
  
  OpenAL          
  
  PDF
  
  Audio Mixing   
  
  Quartz (2D)
  
  Audio Recording
  
  Core Animation
  
  Video Playback 
  
  OpenGL ES

### Core Services

  Collections    
  
  Core Location
  
  Address Book   
  
  Net Services 
  
  Networking    
  
  Threading
  
  File Access    
  
  Preferences
  
  SQLite       
  
  URL Utilities

### Core OS
  OSX Kernel    
  
  Power Management
  
  Mach 3.0      
  
  Keychain Access
  
  BSD          
  
  Certificates
  
  Sockets       
  
  File System
  
  Security     
  
  Bonjour

## Platform Components
### Tools
 Xcode 7.1       
 
 Instr uments
 
### Language(s)
  let value = formatter.numberFromString(display.text!)?.doubleValue
### Frameworks
   Foundation   
   
   Core Data   
   
   UIKit   
   
   Map Kit   
   
   Core Motion
   
### Design Strategy
   MVC
   
## Demo
### Calculator
All this stuff can be very abstract until you see it in action.

We'll start getting comfortable with Swift and Xcode 6 by building something right away.

Two part demo starting today, finishing on Wednesday.

### Today's topics in the demo …

- Creating a Project in Xcode 6
- Building a UI (and making it squishable/stretchable using Autolayout)
- The iOS Simulator
- println (and conversion to a String using \() notation)
- Defining a class in Swift, including how to specify instance variables and methods
- Connecting properties (instance variables) from our Swift code to the UI (outlets)
- Connecting UI elements to invoke methods in our Swift code (actions)
- Accessing iOS documentation from our code
- Optionals (?, unwrapping implicitly by declaring with !, and unwrapping explicitly with ! and if let)

## swift
swift是强类型语言，但是有类型推导。swift中有optional类型，于对optional只能有两个值，一个是nil，一个是非nil。
swift支持闭包运算，一个有趣而又简单的应用是在写计算器程序时，会根据不同操作符进行运算。
```swift
        switch operation{
        case "+":
            if operandStack.count >= 2{
                displayValue = operandStack.removeLast() + operandStack.removeLast()
                enter()
            }
        case "-":
            if operandStack.count >= 2{
                displayValue = operandStack.removeLast() - operandStack.removeLast()
                enter()
            }
        case "*":
            if operandStack.count >= 2{
                displayValue = operandStack.removeLast() * operandStack.removeLast()
                enter()
            }
        case "/":
            if operandStack.count >= 2{
                displayValue = operandStack.removeLast() / operandStack.removeLast()
                enter()
            }
        default :break
```
这是一段糟糕的代码，里面有大量重复的内容，不易于阅读和维护。我们发现四个case分支中几乎都在做同一件事。我们可以定义performOperation函数，这个函数传入的变量是另一个函数，如此可以提高代码可读性。
```swift
switch operation{
        case "+": performOperation(plus)
        case "-": performOperation(minus)
        case "*": performOperation(multiply)
        case "/": performOperation(divide)
        default :break
        }
        
        func performOperation(operation:(Double, Double) ->Double){
            if operandStack.count >= 2{
                displayValue = operation(operandStack.removeLast(),operandStack.removeLast())
                enter()
            }
        }
        func plus(op1: Double, op2: Double) ->Double{
            return op1 + op2
        }
```
上面只列出了plus函数，如果将其余三个函数都写上，似乎代码变得更长了，岂不是越优化越糟糕？这时就需要使用swift中的闭包运算（closure）。
可以将
```swift
case "+": performOperation(plus)
func plus(op1: Double, op2: Double) ->Double{
            return op1 + op2
        }
```
变成

```swift
case “＋”: performOperation({(op1:Double, op2:Double) ->Double in return op1 + op2})
```由于swift有强大的类型推导功能(type inference),我们在performOperation函数中已经定义了传入的参数是(Double,Double)，因此在闭包中可以不指明op1、op2的类型，可以简写为如下形式

```swift
case "+": performOperation({(op1,op2) in return op1 + op2})
```

同理performOperation指明了返回值是Double，因此闭包中可以省略return关键字

```swift
case “＋”：performOperation({(op1,op2) in op1 + op2})
```

swift中并不是必须给参数命名，默认用$0、$1代表各个参数，因此以上语句又可以简写成

```swift
case "+": performOperation({$0 + $1})
```
swfit还有一个特性是，最后一个参数可以将它拿出来写， performOperation函数只有一个参数，因此这个参数必然是最后一个参数，将它移至外边改写成

```swift
case "+": performOperation(){$0 + $1}
```
最后完整的代码如下
```swift
class ViewController: UIViewController {

    @IBOutlet weak var display: UILabel!
    
    var userInTheMiddleOfTypingNumber: Bool = false
    
    @IBAction func operation(sender: UIButton) {
        let digit = sender.currentTitle!
        if userInTheMiddleOfTypingNumber {
        display.text = display.text! + digit
        }
        else {
            display.text = digit
            userInTheMiddleOfTypingNumber = true
        }
    }
    
    var operandStack: Array<Double> = Array<Double>()
 
    
    @IBAction func enter() {
        userInTheMiddleOfTypingNumber = false
        operandStack.append(displayValue)
        print("operandStack\(operandStack)")
        
    }
    
    var displayValue: Double {
        get{
            return NSNumberFormatter().numberFromString(display.text!)!.doubleValue
        }
        set{
            display.text = "\(newValue)"
            userInTheMiddleOfTypingNumber = false
        }
    }
    
    
    
    @IBAction func operateDigit(sender: UIButton) {
        let operate = sender.currentTitle!
        if userInTheMiddleOfTypingNumber{
            enter()
        }
        switch operate{
            case "×": performOperation{ $0 * $1 }
            case "−": performOperation{ $1 - $0 }
            case "+": performOperation{ $0 + $1 }
            case "÷": performOperation{ $1 / $0 }
            case "√": performOperation1{ sqrt($0) }
        default: break
        }
    }
    
    func performOperation(operate:(Double, Double) -> Double){
        if operandStack.count >= 2{
            displayValue = operate(operandStack.removeLast(), operandStack.removeLast() )
            enter()
        }
    }

    func performOperation1(operate: Double -> Double){
        if operandStack.count >= 1{
            displayValue = operate(operandStack.removeLast() )
            enter()
        }
    }


}
```

另外值得一提的是，变量var displayValue手动设置get、set之后方便的将它与display.text UI绑定。swift中变量用var定义，常量用let定义。

## Auto Layout
本课还简要介绍了auto layout，之前的iOS开发大多采用绝对布局。随着近年来iOS设备尺寸的多样化，为了适配不同分辨率屏幕，苹果公司推出了这项技术,auto layout的核心是constraint。


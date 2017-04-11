title: Developing iOS 8 Apps with Swift 学习笔记5
date: 2015-11-08 23:00:00
categories: iOS
tags:
---
lecture 6.
   
<!-- more -->


在UIView Class之前加入@IBDesignable就可以在storyboard看到UIView中的画图

@IBInspectable可以在xcode中可视化修改属性

添加手势
1.向UIView中添加手势
```swift
    @IBOutlet weak var faceView: FaceView!{
        didSet{
            faceView.dataSource = self
            faceView.addGestureRecognizer(UIPinchGestureRecognizer(target: faceView, action: "scale:"))
        }
    }
```
在FaceView中实现scale方法即可

2.想UIViewController中添加手势
storyboard中拖入手势，按住control连接即可
```swift
    @IBAction func changeHappiness(gesture: UIPanGestureRecognizer) {
        switch gesture.state{
        case .Ended: fallthrough
        case .Changed:
            let translation = gesture.translationInView(faceView)
            let happinessChange = -Int(translation.y / Constants.HappinessGestureScale)
            if happinessChange != 0{
                happiness += happinessChange
                gesture.setTranslation(CGPointZero, inView: faceView)
            }
        default:break
        }
    }
```
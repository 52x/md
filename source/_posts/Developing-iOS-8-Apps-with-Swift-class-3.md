title: Developing iOS 8 Apps with Swift 学习笔记3
date: 2015-10-30 11:47:00
categories: iOS
tags:
---
lecture 3. Applying MVC
   
<!-- more -->

这节课都是demo演示，没有lecture。 

完善计算器程序
mvc
enum
simple initializer
tuples
dictionary
return optional

文件名和类名最好相同，所有单词首字母大写。函数、变量等名称量首字母小写，后面单词首字母大写。

var a = Array<Double>() 或者 var a = [Double]()
var b = Dictionary<String,Double> 或者 var b = [Stirng:Double]()

有时想返回一种类型的值，但是又无法返回，可以使用optional。例如evaluate正常情况要返回Double，用户操作错误时返回nil。Duoble与nil不统一，可以用optional。
func evaluate()->Duoble?{
}

array和dictionary在swift中不是class是struct。class传递的十引用，struct按值传递。

swift的copy使用了copy on write技术。

```swift
//
//  CalculatorBrain.swift
//  Calculator
//
//  Created by admin on 15/11/2.
//  Copyright © 2015年 huic. All rights reserved.
//

import Foundation

class CalculatorBrain
{
    private enum Op : CustomStringConvertible{
        case Operand(Double)
        case UnaryOperation(String,Double->Double)
        case BinaryyOperation(String,(Double,Double)->Double)
        
        var description:String{
            get{
                switch self{
                case .Operand(let operand):
                    return "\(operand)"
                case .UnaryOperation(let symbol, _):
                    return symbol
                case .BinaryyOperation(let symbol, _):
                    return symbol
                }
            }
        }
    }
    
    private var opStack = [Op]()
    private var knowOps = [String:Op]()
    
    init(){
        knowOps["×"] = Op.BinaryyOperation("×", *)
        knowOps["÷"] = Op.BinaryyOperation("÷"){ $1 / $0 }
        knowOps["+"] = Op.BinaryyOperation("+", +)
        knowOps["-"] = Op.BinaryyOperation("-"){ $1 - $0 }
        knowOps["√"] = Op.UnaryOperation("√",sqrt)
    }
    
    private func evaluate(ops:[Op])->(result:Double?,remainingOps:[Op]){
        if !ops.isEmpty{
            var remainingOps = ops
            let op = remainingOps.removeLast()
            switch op{
            case .Operand(let operand):
                return (operand,remainingOps)
            case .UnaryOperation(_, let operation):
                let operandEvaluation = evaluate(remainingOps)
                if let operand = operandEvaluation.result{
                    return (operation(operand),operandEvaluation.remainingOps)
                }
            case .BinaryyOperation(_, let operation):
                let op1Evaluation = evaluate(remainingOps)
                if let operand1 = op1Evaluation.result{
                    let op2Evaluation = evaluate(op1Evaluation.remainingOps)
                    if let operand2 = op2Evaluation.result{
                        return (operation(operand1,operand2),op2Evaluation.remainingOps)
                    }
                }
            }
            
        }
        return (nil,ops)
    }
    func evaluate()->Double?{
        let(result, remainder) = evaluate(opStack)
        print("\(opStack)=\(result) with \(remainder) left over")
        return result
    }
    
    func pushOperand(operand: Double)->Double?{
        opStack.append(Op.Operand(operand))
        return evaluate()
    }
    
    func performOperation(symbol: String)->Double?{
        if let operation = knowOps[symbol]{
            opStack.append(operation)
        }
        return evaluate()
    }
}
```

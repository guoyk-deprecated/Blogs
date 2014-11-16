---
layout: post
title: "[Swift] 自定义运算符"
---

我不得不说苹果这帮人太机智了，直到今天我才发现`Swift`的自定义运算符这么好用。

举个栗子，`UIBezierPath`，用过的人都知道有三个方法最常用。

*  `moveToPoint:`
*  `addLineToPoint:`
*  `addCurveToPoint:controlPoint1:controlPoint2:`

然后，我们可以自己定义三个运算符来快捷调用这三个方法

```swift
// 首先，声明这三个运算符，如果之前声明过，应该不会冲突

infix operator --> { associativity left }   
infix operator +-> { associativity left }
infix operator +~> { associativity left }

// 然后，声明运算符到底怎么干活

func -->(left: UIBezierPath, right: [CGFloat]) -> UIBezierPath {
      left.moveToPoint(CGPointMake(right[0], right[1]))
        return left
}

func +->(left: UIBezierPath, right: [CGFloat]) -> UIBezierPath {
      left.addLineToPoint(CGPointMake(right[0], right[1]))
        return left
}

func +~>(left: UIBezierPath, r: [[CGFloat]]) -> UIBezierPath {
      left.addCurveToPoint(CGPointMake(r[0][0], r[0][1]), controlPoint1: CGPointMake(r[1][0], r[1][1]), controlPoint2: CGPointMake(r[2][0],r[2][1]))
        return left
}
```

当当！我们就可以以这种风骚的语法来构建一个`UIBezierPath`了

```swift
let path = UIBezierPath()
path --> [69,2]                              // 移动到一个点
 +-> [36.5, 9.5]                             // 画条直线到一个点
 +~> [[16, 82], [-9.5, 29.5], [13.5, 79]]    // 画条曲线到一个点，外带两个控制点
```

再看看以前的写法，简直弱爆了:

```swift
let path = UIBezierPath()
path.moveToPoint(CGPointMake(11,23))
path.addLineToPoint(CGPointMake(12,33))
path.addCurveToPoint(CGPointMake(12,33), controlPoint1: CGPointMake(0, 5), controlPoint2: CGPointMake(40, 22))
```

`associativity left` 这句话声明了我们的自定义运算符是向左结合的，也就是说，这几个运算符可以一直连下去。

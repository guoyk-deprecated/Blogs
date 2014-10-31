---
layout: post
title: Swift Custom Operators
---

I have to say Apple guys are fxxking awesome.

It took me so long time to realize `Custom Operators` are sooooooooooo useful.

For example, `UIBezierPath`, there are three methods we use most frequently.

*  `moveToPoint:`
*  `addLineToPoint:`
*  `addCurveToPoint:controlPoint1:controlPoint2:`

We can just define three custom operators like this:

```swift
// declare three operators, with associativity left. won't conflict if they are defined before

infix operator --> { associativity left }   
infix operator +-> { associativity left }
infix operator +~> { associativity left }

// declare how custom operators work with UIBezierPath

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

Voila, now we can build a `UIBezierPath` like this:

```swift
let path = UIBezierPath()
path --> [69,2]                              // Move to point
 +-> [36.5, 9.5]                             // Add a line to point
 +~> [[16, 82], [-9.5, 29.5], [13.5, 79]]    // Add a curve to a point, with two control points
```

other than this:

```swift
let path = UIBezierPath()
path.moveToPoint(CGPointMake(11,23))
path.addLineToPoint(CGPointMake(12,33))
path.addCurveToPoint(CGPointMake(12,33), controlPoint1: CGPointMake(0, 5), controlPoint2: CGPointMake(40, 22))
```

These operations are totoally associative, it means you can chain them as many as you can.

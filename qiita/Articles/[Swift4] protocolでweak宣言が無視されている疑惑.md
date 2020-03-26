Swift4.0でこんな実装をしていた。

```swift
protocol HogeView: class {
    weak var delegate: HogeViewDelegate? { get set }
}

protocol HogeViewDelegate: class {
    // delegate functions
}

class HogeViewController: UIViewController, HogeView {
    weak var delegate: HogeViewDelegate?

    // 以下VC実装
}
```

ところが、このプロジェクトをXcode9.4.1で開いたところ

`'weak' should not be applied to a property declaration in a protocol and will be disallowed in future versions`

という警告が出るように

調べてみると、
[[SE-0186] Remove ownership keyword support in protocols](https://github.com/apple/swift-evolution/blob/master/proposals/0186-remove-ownership-keyword-support-in-protocols.md)
ということで`protocol`で`weak`なプロパティが宣言できなくなる模様。

そこでちょっと気になったので
<details><summary>Swift4.0でこんなコードを書いてみた。</summary><div>

```swift
//: Playground - noun: a place where people can play

import Foundation

class A {}
extension A: CustomDebugStringConvertible {
    var debugDescription: String {
        return "A(\(Unmanaged.passUnretained(self).toOpaque()))"
    }
}

protocol P: class {
    weak var weakVar: A? { get set }
}

class B: P {
    weak var weakVar: A?
}
extension B: CustomDebugStringConvertible {
    var debugDescription: String {
        return "B(\(Unmanaged.passUnretained(self).toOpaque()))"
    }
}

var b: B = B()
print(b, b.weakVar.debugDescription)

b.weakVar = A()
print(b, b.weakVar.debugDescription)

print()

var p: P = B()
print(p, p.weakVar.debugDescription)

p.weakVar = A()
print(p, p.weakVar.debugDescription)
```

</div></details>

<details><summary>結果</summary><div>

```console
B(0x0000600000030600) nil
B(0x0000600000030600) nil

B(0x00006000000335c0) nil
B(0x00006000000335c0) Optional(A(0x0000600000008530)) <- !?
```
</div></details>

`P`型で宣言した変数の`weakVar`が`weak`になっていない...？

そう考えるとSwift4.1で`protocol`で`weak`宣言できなくなっても動きとしては変わらなそうだがはて... :thinking: 

# 追記

```swift
class A {
    deinit {
        print("deinit A")
    }
}

// 中略

var b: B = B()
print(b, b.weakVar.debugDescription)

b.weakVar = A()
print(b, b.weakVar.debugDescription)

print()

var obj: Any = b
if let p = obj as? P {
    print(p, p.weakVar.debugDescription)
    p.weakVar = A()
    print(p, p.weakVar.debugDescription)
}

print(b, b.weakVar.debugDescription)
```

<details><summary>結果</summary><div>

```console
B(0x00006000000273e0) nil
deinit A
B(0x00006000000273e0) nil

B(0x00006000000273e0) nil
B(0x00006000000273e0) Optional(A(0x0000604000003f10))
deinit A
B(0x00006000000273e0) nil
```
</div></details>

`protocol`でキャストしている間だけretainされている(スコープ抜けたらreleaseされてる)ようなので、循環参照問題は大丈夫と見ていいのかなー

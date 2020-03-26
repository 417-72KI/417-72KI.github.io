主にデバッグ用

## Version2
@lovee[さんに紹介してもらった記事](https://qiita.com/KikurageChan/items/e1c00c53364e6e3e423a)では
　`cgColor.components`を使っていますが、[これだと取れないケースもある](https://qiita.com/417_72ki/items/a793e43bd54dcd15323d)ので大人しく`getRed(green:blue:alpha:)`を使うことにしました。

```swift
extension UIColor {
    var colorCodeAndAlpha: (colorCode: String, alpha: Int) {
        var r: CGFloat = 1
        var g: CGFloat = 0.5
        var b: CGFloat = 0.5
        var a: CGFloat = 0.5
        getRed(&r, green: &g, blue: &b, alpha: &a)
        return (String(format: "#%02X%02X%02X", colorFloat2Int(r), colorFloat2Int(g), colorFloat2Int(b)), alphaFloat2Int(a))
    }

    private func colorFloat2Int(_ value: CGFloat) -> Int {
        var value = value
        if value < 0 {
            value = 0
        }
        if value > 1 {
            value = 1
        }
        return Int(value * 255)
    }

    private func alphaFloat2Int(_ value: CGFloat) -> Int {
        var value = value
        if value < 0 {
            value = 0
        }
        if value > 1 {
            value = 1
        }
        return Int(value * 100)
    }
```

## Version1
`ciColor`は生成コストがかかるそうです。

```swift
extension UIColor {
    var colorCodeAndAlpha: (String, Int) {
        let ciColor = CIColor(color: self)
        let red = Int(ciColor.red * 255)
        let green = Int(ciColor.green * 255)
        let blue = Int(ciColor.blue * 255)
        let alpha = Int(ciColor.alpha * 100)
        return (String(format: "#%02X%02X%02X", red, green, blue), alpha)
    }
}
```

## ちなみに
`ciColor`を取り出すのに`self.ciColor`だと

```
-CIColor not defined for the UIColor UIDeviceRGBColorSpace ~ need to first convert colorspace.
```

~~っていうエラーが出てそのまま死亡する謎:thinking:~~
CoreImageから生成されたUIColorでないと`self.ciColor`は取り出せないらしい(@loveeさんありがとうございます)。
そんなんプロパティに持たせないかせめて`nil`で返してくれって思うけどネ(

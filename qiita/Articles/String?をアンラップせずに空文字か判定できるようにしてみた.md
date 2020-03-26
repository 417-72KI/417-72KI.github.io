
## 空文字判定の前にnull判定

文字列の空判定ってどの言語でも出て来ると思うんだけど、大体null判定した後に中身が空文字かどうかを精査しないといけない。


## 特にSwiftだと
型が`String`なら

```
if !str.isEmpty {
	// 処理
}
```

で済む話なんだけど、`String?`だと

```
if str != nil && !str!.isEmpty {
	// 処理
}
```

ってやって ***Force Unwrap*** が出てくるのが気に食わない。(っていうか **Java** かよ←

## if letを使って


```
if let str = str, !str.isEmpty {
	// 処理
}
```

って書くこともできて、アンラップした値を中の処理で使うなら良いんだけど使わない場合なんかもにょる。

## そこで

(´-\`).｡oO(`Optional`の`extension`作ってやればよくね？

<details><summary>ってことでできたコードがこちら</summary><div>

```swift:StringExtension.swift
protocol StringOptionalProtocol {}
extension String: StringOptionalProtocol {}

extension Optional where Wrapped: StringOptionalProtocol {
    
    /// nilまたは空文字の場合trueを返す
    var isEmpty: Bool {
        if let str = self as? String {
            // アンラップに成功したので空文字か判定
            return str.isEmpty
        }
        
        // アンラップに失敗してるのでnilと判定
        return true
    }
    
    /// nilまたは空文字の場合falseを返す
    var isNotEmpty: Bool {
        guard let str = self as? String else {
            // アンラップに失敗しているのでnilと判定
            return false
        }
        
        // アンラップに成功したので空文字でないか判定
        return !str.isEmpty
    }
}
```

`String`に専用のプロトコルを用意してジェネリクスの型引数(`Wrapped`)に制約をかけているのがミソ。
こうすることで`String?`に対してだけこの中のメソッドを許可できる。
後は中で`isEmpty`と`isNotEmpty`を実装してやればOK
</div></details>

## ※2019/7/30追記
ずっと放置していて忘れてたんですが、Swift4くらいのタイミングから`where`で直接型を指定できるようになっていました。
<details><summary>それに基づいて書き直したのがこちら</summary><div>

```swift:StringExtension.swift
extension Optional where Wrapped == String {
    /// nilまたは空文字の場合trueを返す
    var isEmpty: Bool {
        guard let str = self else {
            // アンラップに失敗してるのでnilと判定
            return true
        }

        // アンラップに成功したので空文字か判定
        return str.isEmpty
    }

    /// nilまたは空文字の場合falseを返す
    var isNotEmpty: Bool {
        guard let str = self else {
            // アンラップに失敗しているのでnilと判定
            return false
        }

        // アンラップに成功したので空文字でないか判定
        return !str.isEmpty
    }
}
```
</div></details>

## 呼び出し例はこんな感じ

```
func hoge(_ str: String?) {
	if str.isEmpty {
		print("empty string")	
	}
}

hoge("hoge")	// 何も出力されない
hoge("")		// "empty string"が出力される
hoge(nil)		// "empty string"が出力される

```

## 注意

- 変数が`nil`の時と`""`の時で処理が変わる場合はこの方法は無意味なので用法用量を守って正しく使いましょう(

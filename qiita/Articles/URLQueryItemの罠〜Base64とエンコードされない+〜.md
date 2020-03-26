`URLComponents`と`URLQueryItem`を使うことでGETパラメータのURL(%)エンコードをよしなにやってくれるようになってから久しいですが、何も考えずにそれで実装した結果痛い目に遭ったお話。

# やろうとしたこと
AES+CBCで暗号化した文字列と初期化ベクトルをbase64にしてクエリパラメータで送る。

# 当初の実装
```swift
func createURL(text: String, iv: String) -> URL? {
    var comps = URLComponents(string: "https://hoge.fuga")
    let queryItems = [
        URLQueryItem(name: "text", value: text),
        URLQueryItem(name: "iv", value: iv)
    ]
    comps?.queryItems = queryItems
    return comps?.url
}
```

# 何が起きた？
通信先で復号に失敗するケースが発生(発生頻度はランダム)。
ちなみに端末側では100%復号に成功する。

# 原因
復号に成功するパターンもあったため原因の特定に時間がかかりましたが、
失敗するパターンに共通していたのが、**base64化した文字列に`+`が含まれる**ということでした。

詳細は[こちら](https://www.glamenv-septzen.net/view/1170)の記事を見ると早いと思いますが、どうやらサーバ側でこの`+`を半角スペースと認識した結果文字列自体が壊れていたようです。

また、`URLComponents`は**RFC3986**に準拠しているため、`+`をエンコードしてくれません。
そのため、`URLQueryItem`に`+`を含んだ暗号化文字列/初期化ベクトルをそのまま突っ込むと壊れたデータとしてエラーが返ってくるという構図になっていました。

# 対策後の実装

```swift
var characterSet: CharacterSet {
    var set = CharacterSet.urlQueryAllowed
    set.remove("+")
    return set
}

func createURL(text: String, iv: String) -> URL? {
    var comps = // 略
    let queryItems = // 略
    comps?.queryItems = queryItems
    // 生成された生クエリを取り出す
    let query = comps?.query
    // ↑を手動でエンコードしたものをエンコード済みクエリとしてセットする
    comps?.percentEncodedQuery = query?.addingPercentEncoding(withAllowedCharacters: characterSet)
    return comps?.url
}
```

`comps?.percentEncodedQuery`を直接取り出して`replacingOccurrences(of: "+", with: "%2B")`したものを再度`comps?.percentEncodedQuery`に放り込んでもよかったのですが、`"%2B"`をハードコーディングしたくなかったので(`"+"`は良いのかっていうツッコミは置いといて)↑の形に落ち着きました。

# ちなみに
iOS11から`URLComponents`に`percentEncodedQuery`というプロパティが加わり、
予めエンコードした値をセットした`URLQueryItem`を放り込めるようになりました。

これを使うと以下の通りスッキリ書けるようになります。

```swift
// 前略
let encodedItems = queryItems.map { URLQueryItem(name: $0.name, value: $0.value?.addingPercentEncoding(withAllowedCharacters: set)) }
comps?.percentEncodedQueryItems = encodedItems
return comps?.url
```

更に、`URLQueryItem`にextensionを生やして

```swift
extension URLQueryItem {
    func addingPercentEncoding(withAllowedCharacters characterSet: CharacterSet) -> URLQueryItem {
        return URLQueryItem(name: name, value: value?.addingPercentEncoding(withAllowedCharacters: characterSet))
    }
}
```
```swift
// 前略
let encodedItems = queryItems.map { $0.addingPercentEncoding(withAllowedCharacters: set) }
comps?.percentEncodedQueryItems = encodedItems
return comps?.url
```

ともできますね。

# 参考
- https://www.glamenv-septzen.net/view/1170
- https://stackoverflow.com/questions/43052657/encode-using-urlcomponents-in-swift
- https://code.i-harness.com/en/q/1e1d464

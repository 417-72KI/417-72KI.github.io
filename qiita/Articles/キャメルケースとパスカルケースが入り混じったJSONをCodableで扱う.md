最近のRestfulなAPIなら無いと思いますが、業務で別の会社がサーバを担当するケースでは稀にそういった~~クソみたいな~~APIにブチ当たる場合があります~~(というか現在進行系でぶち当たってて頭を抱えてる状態😂)~~。

例えばこんな感じ

```json
{
    "ResponseHeader": {
        "status": "0",
        "message": "",
        "validationErrorMessages": []
    },
    "itemList": [
        {
            "name": "hoge"
        }
    ]
}
```

~~いやもう`ResponseHeader`ってなんやねんってツッコミは置いといて、~~
こういうJSONを`Codable`で扱おうとすると`CodingKey`を用意しないといけなくて面倒です。

```swift:Response.swift
struct Response: Codable {
    let responseHeader: ResponseHeader
    let itemList: [Item]

    enum CodingKeys: String, CodingKey {
        case responseHeader = "ResponseHeader"
        case itemList
    }
}
```

しかし、`JSONDecoder`の`keyDecodingStrategy`を使えばなんとかできるのではなかろうかと思って検証してみました。

`JSONEncoder.KeyEncodingStrategy.custom(@escaping ([CodingKey]) -> CodingKey)`については[Appleのドキュメント](https://developer.apple.com/documentation/foundation/jsondecoder/keydecodingstrategy/custom)と[この記事](https://qiita.com/_ha1f/items/bf1aad5ea3e927f59f9d#custom)が参考になります。
↑を元にまずは`ResponseKey`を作ります。

```swift:ResponseKey.swift
struct ResponseKey: CodingKey, ExpressibleByStringLiteral {
    typealias StringLiteralType = String

    var stringValue: String
    var intValue: Int?

    init(stringLiteral value: String) {
        self.stringValue = value
        self.intValue = nil
    }

    init?(stringValue: String) {
        self.stringValue = stringValue
        self.intValue = nil
    }

    init?(intValue: Int) {
        self.stringValue = String(intValue)
        self.intValue = intValue
    }
}
```

そして`JSONDecoder`をこんな感じで設定してあげます。

```swift:decoder.swift
var decoder: JSONDecoder {
    let decoder = JSONDecoder()
    decoder.keyDecodingStrategy = .custom { keys in
        // 先頭文字を小文字にする
        let key = keys.last!.stringValue.lowercasingFirst
        return ResponseKey(stringLiteral: key)
    }
    return decoder
}
```

こうすることで、この`decoder`を使ってデコードするJSONは全てのキーを先頭小文字に変換して扱ってくれるようになります。
つまり、キャメルケースであろうとパスカルケースであろうと問答無用でキャメルケースとして扱うわけです。

これで、上の`Response`は`CodeingKey`を定義することなく、シンプルに

```swift:Response.swift
struct Response: Codable {
    let responseHeader: ResponseHeader
    let itemList: [Item]
}
```

と書くことができるようになります。

ちなみに`JSONDecoder.KeyDecodingStrategy.custom`で用意するクロージャがなぜ`[CodingKey] -> CodingKey`なのかはドキュメントを読んでみてもいまいちピンと来ませんでした。

なぜだ...🤔

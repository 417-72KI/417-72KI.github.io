Swift2.x系で書かれていた既存のプロジェクトをSwift4へとコンバートしていたら、

```swift
var flag = true
    
func action(_ flag: inout Bool, completion: () -> Void) {
    flag = !flag
    completion()
}

action(&flag) {
    log.debug(flag) // ここでクラッシュ
}
```

というようなコードで

```console
Simultaneous accesses to ~, but modification requires exclusive access.
Previous access (a modification) started at ~.
```

というメッセージが出てクラッシュするようになりました。

どうやらSwift4ではメモリへの排他アクセスが強制されるようになり、
`inout`なパラメータに渡したポインタを中から参照しようとすると
同じメモリへの同時アクセスとみなされクラッシュしていた模様。

# 回避策

`TARGETS -> Build Settings -> Swift Compiler - Code Generation`
の
`Exclusive Access to Memory`
を
`Full Enforcement (Run-time Checks in Debug Builds Only)`
から
`No Enforcement`
に変更するとクラッシュしなくなります。

本当は同時アクセスにならないようにコード自体を見直す必要がありますが、
古いSwiftからのコンバートという点を鑑みた結果、上記の対応でお茶を濁すことにしました(

Swift4で新規アプリを開発する際はしっかり上記を意識した実装をする必要がありますね。

# 参考
[Conflicting Access to In-Out Parameters](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/MemorySafety.html#//apple_ref/doc/uid/TP40014097-CH46-ID571)
[Enforce Exclusive Access to Memory](https://github.com/apple/swift-evolution/blob/master/proposals/0176-enforce-exclusive-access-to-memory.md)

# TL;DR

```swift
hoge.subscribe(
    onNext: { ~ },
    onError: { ~ }
)
.disposed(by: disposeBag)
fuga.subscribe(
    onNext: { ~ },
    onError: { ~ }
)
.disposed(by: disposeBag)
```

を

```swift
disposeBag.insert(
    hoge.subscribe(
        onNext: { ~ },
        onError: { ~ }
    ),
    fuga.subscribe(
        onNext: { ~ },
        onError: { ~ }
    )
)
```

って書こうぜって話

# `DisposeBag`とは
subscribeしている`Observable`等をまとめてdisposeしてくれる仕組みです。
`DisposeBag`オブジェクトが破棄されるタイミングで抱えている`Disposable`に対して`dispose()`をかけるようになっています。

参考記事は[こちら](http://harumi.sakura.ne.jp/wordpress/2019/02/05/dispose%E3%81%A8disposedby-disposebag%E3%82%92%E4%BD%BF%E3%81%84%E3%81%93%E3%81%AA%E3%81%99/)

# `DisposeBag`の実装
ソース
https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposables/DisposeBag.swift

まずは`disposed(by:)`を見ていきます

```swift:DisposeBag.swift
extension Disposable {
    /// Adds `self` to `bag`
    ///
    /// - parameter bag: `DisposeBag` to add `self` to.
    public func disposed(by bag: DisposeBag) {
        bag.insert(self)
    }
}
```

はい、`bag`に`insert`してるだけですねｗ
ではこの`insert`を見ていきます

```swift:DisposeBag.swift
private var _lock = SpinLock()

// state
private var _disposables = [Disposable]()
private var _isDisposed = false

/// Adds `disposable` to be disposed when dispose bag is being deinited.
///
/// - parameter disposable: Disposable to add.
public func insert(_ disposable: Disposable) {
    self._insert(disposable)?.dispose()
}

private func _insert(_ disposable: Disposable) -> Disposable? {
    self._lock.lock(); defer { self._lock.unlock() }
    if self._isDisposed {
        return disposable
    }
    self._disposables.append(disposable)
    return nil
}
```

これを見ると、
- `bag`が既に破棄されている場合には`insert`された`Disposable`も破棄しちゃう
- `bag`が破棄されていないなら内部のキューに積む

といった動きをしていることが分かります。

また、`DisposeBag`には複数の`Disposable`をまとめて処理するためのメソッドも用意されています。

```swift:DisposeBag.swift
/// Convenience function allows a list of disposables to be gathered for disposal.
public func insert(_ disposables: Disposable...) {
    self.insert(disposables)
}

/// Convenience function allows an array of disposables to be gathered for disposal.
public func insert(_ disposables: [Disposable]) {
    self._lock.lock(); defer { self._lock.unlock() }
    if self._isDisposed {
        disposables.forEach { $0.dispose() }
    } else {
        self._disposables += disposables
    }
}
```

こちらも

- `bag`が既に破棄されている場合には`insert`された`Disposable`も破棄しちゃう
- `bag`が破棄されていないなら内部のキューに積む

という点では一緒ですね。

つまり、

```swift
hoge.subscribe(
    onNext: { ~ },
    onError: { ~ }
)
.disposed(by: disposeBag)
fuga.subscribe(
    onNext: { ~ },
    onError: { ~ }
)
.disposed(by: disposeBag)
```

といったコードは

```swift
disposeBag.insert(
    hoge.subscribe(
        onNext: { ~ },
        onError: { ~ }
    )
)

disposeBag.insert(
    fuga.subscribe(
        onNext: { ~ },
        onError: { ~ }
    )
)
```

と置き換えられ、更に

```swift
disposeBag.insert(
    hoge.subscribe(
        onNext: { ~ },
        onError: { ~ }
    ),
    fuga.subscribe(
        onNext: { ~ },
        onError: { ~ }
    )
)
```

と置き換えられることが分かります。

見た目的にもここでまとめて`disposeBag`に突っ込んでることが分かりやすくなりますし、
`SpinLock`については詳しくないので断言はできませんが、
`.disposed(by:)`だと一々ロックかけたり解除したりを繰り返すのに対し、
まとめて`insert`するなら1回のロックで済むのでパフォーマンス的にも良いんじゃないかなーと思ってます。
(あと、`.`繋ぎは改行するような規約の現場なら、シンプルに行数が減るのでLinterにも優しくなるはず)

# 結論
`DisposeBag.insert()`、積極的に使っていこう！

# 参考
https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposables/DisposeBag.swift

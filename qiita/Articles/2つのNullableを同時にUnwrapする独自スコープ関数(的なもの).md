
# 前置き
Kotlinといえばスコープ関数。
そのスコープ関数の中でも一番利用頻度が高いのが `let` による`Nullable` のアンラップですね。

```Kotlin
fun doSomething(a: String) {
	print(a)
}

fun hoge(a: String?) {
	a?.let { doSomething(it) } // a == null の場合は表示されない
}
```

これは

```Kotlin
if(a != null) {
	doSomething(a) // aはsmart castによりStringとして扱われる
}
```

と書くこともできますが、オブジェクトのNullableなプロパティ等に対して使うことはできません。

```Kotlin
class Hoge {
    var a: String? = null
}

fun doSomething(hoge: Hoge) {
    if (hoge.a != null) {
        fuga(hoge.a) // コンパイルエラー "Smart cast to 'String' is impossible, because 'hoge.a' is a mutable property that could have been changed by this time"
    }
}

fun fuga(str: String) {
    print(str)
}
```

`hoge.a != null` の判定を行ってから `fuga(hoge.a)` を呼ぶまでの間に何らかの要因(別スレッドから書き換えられる等)で `hoge.a` が `null` になってしまう可能性があるからキャストできないよ、ということです。

当然、こういう場合は以下のようにするのが最適解ですね。

```Kotlin
hoge.a?.let { fuga(it) } // ① hoge.aをアンラップした値はitでそのまま取り出せる
hoge.a?.let { a -> fuga(a) } // ② もちろんパラメータ名を指定することもできる
hoge.a?.let(::fuga) // ③ 関数参照(でいいのかな？)で呼び出す
```

# 本題

さて、こういうケースを考えてみましょう。

```Kotlin
fun doSomething(a: String, b: String) {
    // do something
}
```

この `doSomething` をNullableな2つの値で呼び出すことを考えます。

```Kotlin
class Hoge {
    var a: String? = "a"
    var b: String? = null
}
```

もちろん、

```Kotlin
if (hoge.a != null && hoge.b != null) {
    doSomething(a, b)
}
```

と書いても先述の通りsmart castが効かないのでNGです。
かと言って、

```Kotlin
hoge.a?.let { a ->
    hoge.b?.let { b ->
        doSomething(a, b)
    }
}
```

と書くのも無意味なネストができるだけ...

# そこで
こんな関数を定義してみました。

```Kotlin
inline fun <T1, T2, R> T1.letWith(other: T2?, action: (T1, T2) -> R): R?
    = other?.let { action(this, other) }
```


## 使い方
こんな風に使います。

```Kotlin
hoge.a?.letWith(hoge.b) { a, b -> doSomething(a, b) }
```

こうすることで、 `hoge.a` と `hoge.b` がどちらも `null` じゃない時にだけクロージャの中を実行する、という動作を実現できます。
もちろん、

```Kotlin
val str = hoge.a?.letWith(hoge.b) { a, b -> "a: $a, b: $b" } ?: "(empty)"
print(str)
/*
 * hoge.a != null && hoge.b != null → a: $a, b: $b
 * hoge.a == null || hoge.b == null → (empty)
 */
```

のようにelvis演算子を使うこともできます。

2変数の同時アンラップで悩まされている方、ご参考にしてみてはいかがでしょうか？

# 注意
お気づきの方もいるとは思いますが、これは `hoge.a` と `hoge.b` のどちらが `null` かを判定する必要がある場合には使えません。
あくまでどちらも `null` でない時だけ特定の処理をしたいという場合を想定しています。


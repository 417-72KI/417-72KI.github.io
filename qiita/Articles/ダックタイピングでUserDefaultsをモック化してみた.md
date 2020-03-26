# はじめに
iOSDC2019 Day 0のTrack Eにて登壇してきました。
登壇報告も兼ねて記事にします。
プロポーザルは[こちら](https://fortee.jp/iosdc-japan-2019/proposal/873b4cdb-4c92-4111-bf0b-67a67dbb242e)
スライドは[こちら](https://speakerdeck.com/417_72ki/datukutaipingutoiddeuserdefaultswomotukuhua-suru)

# ダックタイピングとは
ざっくり言うと、メソッドを実行する際にそのオブジェクトの型を見てメソッドを実行できるかを判断するのではなく、その**オブジェクトそのもの**が対象のメソッドを持っているかを判断基準とする考え方です。
RubyやPythonといった動的型付けの言語で使われます。

Ruby界隈で有名なDave Thomasという方がこんな発言をしています
> If it walks like a duck and quacks like a duck, it must be a duck
> (アヒルのように歩き、アヒルのように鳴くならそいつはアヒルのはずだ)

## 静的型付けと動的型付け
静的型付け言語では、コンパイル時に型を見てメソッドが実行できるかを判定します。
例えば、Swiftでこんなコードを書いてみます

```Swift
class Duck {
    func walk() {
        print("Duck walking")
    }
    func quack() {
        print("Quack!!")
    }
}

class Dog {
    func walk() {
        print("Dog walking")
    }
    func bark() {
        print("Bark!!")
    }
}

func walkAndQuack(_ duck: Duck) {
    duck.walk()
    duck.quack()
}

walkAndQuack(Duck())
walkAndQuack(Dog()) // -> Compile Error
```

当たり前ですが最後の行でコンパイルエラーになりますね。

では、Pythonで似たようなコードを書きます

```python
class Duck:
    def walk(self):
        print("Duck walking")
    def quack(self):
        print("Quack!!")
class Dog:
    def walk(self):
        print("Dog walking")
    def bark(self):
        print("Bark!!")

def walk_and_quack(animal):
    animal.walk()
    animal.quack()

walk_and_quack(Duck())
# Duck walking
# Quack!!

walk_and_quack(Dog())
# Dog walking
# AttributeError: 'Dog' object has no attribute 'quack'
```

Pythonでは、引数や変数に型を指定しないため、そのまま実行できます。
そして、実行時にエラーが発生します。
このように、実行時に問題があったときに初めてエラーが出るのが動的型付けの特徴です。

ちなみに、iOSでもSwiftではなくObjective-Cで実装すると同じようなことができます。

```objc
@interface Duck: NSObject
- (void)walk;
- (void)quack;
@end

@implementation Duck
- (void)walk {
    NSLog(@"Duck walking");
}
- (void)quack {
    NSLog(@"Quack!!");
}
@end

@interface Dog: NSObject
- (void)walk;
- (void)bark;
@end

@implementation Dog
- (void)walk {
    NSLog(@"Dog walking");
}
- (void)bark {
    NSLog(@"Bark!!");
}
@end

void walkAndQuack(id duck) {
    [duck walk];
    [duck quack];
}

walkAndQuack([Duck new]);
// Duck walking
// Quack!!

walkAndQuack([Dog new]);
// Dog walking
// unrecognized selector sent to instance
```

Objective-Cを見たこと無い方には見慣れない`id`という型が出てきましたが、これはObjective-C独自の汎用データ型です。Swiftで言う`Any`に該当します。
この型の特徴として、キャストすることなく代入やメソッド呼び出しが可能な点が挙げられます。
これにより、型安全性をある程度捨てて柔軟性にステータスを振っています。

なぜこういうことができるかと言うと、Objective-Cの型システムがCをベースにしているからです。

## 強い型付けと弱い型付け
弱い型付けには
- 実行中に型のチェックをしない
- 型が一致しないオブジェクトを代入しても コンパイルエラーにならない(警告のみ)
- メソッドを呼ぶ際に問題があるとクラッシュする
という特徴があります。
一方、強い型付けでは代入時に型が一致していないとエラーが発生するようになっています。

CやObjective-Cは弱い型付け、KotlinやSwiftは強い型付けに分類されます。

# ダックタイピング + UserDefaults -> MockUserDefaults
弱い型付けの特徴である`メソッドを呼ぶ際に問題があるとクラッシュする`は、逆を言うと `オブジェクトがそのメソッドを実行できればOK`ということになります。
つまり、`NSUserDefaults`が持っているメソッドをそっくり真似たクラスを作って偽装し、`NSUserDefaults`型として扱うことができれば、`UserDefaults`をモック化しなおかつTestableにすることができそうです。

これを実行に移したものをライブラリとしてGitHubに公開しています。
CocoaPods及びCarthageに対応しています。
https://github.com/417-72KI/MockUserDefaults

iOSDCのトークでお見せしていたデモアプリ~~は現在仕上げ中なのでもう少しでmasterにマージできるはずですのでお待ちいただければ...😭~~もmasterにmergeしました！
clone後に`make init_demo_app`でxcodeproj及びxcworkspaceが生成されるようになっています！
XcodeGenを使っているのでそちらも参考になれば嬉しいです😊

# トークについて
iOSDCでのトークは2回目でしたが、やっぱり緊張しまくりでした😅
話している間にも心臓の鼓動音が分かるくらいヤバかったですｗ
なのでちょいちょい深呼吸を挟んで落ち着かせるっていう時間が必要でした😅
去年の初登壇から1年の間にLT会は積極的に参加して場慣れはしているつもりでしたが、30分のセッションは機会が無かったので時間配分も難しく、発表中は半分無我夢中で進めていた気がします。
質疑であった `keyPath`や`dynamicMemberLookup`については別途調査したいと思います😇

聞きに来ていただいた方々、ありがとうございました！

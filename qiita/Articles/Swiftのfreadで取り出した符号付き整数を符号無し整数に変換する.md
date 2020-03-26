
C++で書かれたコードをSwiftにリプレイスする必要があり、
その途中fread周りで詰まったのでメモ

# その前に
## freadって？

ファイルから指定バイト数のデータを指定した数読み込み、バッファに格納する関数

```C++(っていうかC)
size_t fread(void *buf, size_t size, size_t n, FILE *fp); 
```

これがSwiftでも用意されている

```Swift
public func fread(_ __ptr: UnsafeMutableRawPointer!, _ __size: Int, _ __nitems: Int, _ __stream: UnsafeMutablePointer<FILE>!) -> Int
```

## freadの使い方

呼び出しは大体こんな感じ(fpは共にfopenで取り出したファイルポインタ、nは読み込むサイズ)

```C++
char buf[1024];
fread(buf, sizeof(char), n, fp);
```

```Swift
var buffer: [CChar] = [CChar](repeating: 0, count: 1024)
fread(&buffer, Int(MemoryLayout<CChar>.size), n, fp)
```

# 本題

## どこで詰まった？

バッファから数値を取り出す際に、バッファに格納された値をlldbで見てみるとこういう場面に遭遇した

```C++
(lldb) p buf[3]
(char) $2 = '\xd2'
```

```Swift
(lldb) po buf[3]
-46
```

...表示のされ方が違う...だと...？

試しにSwift側で表示された数値を色々変換してみた

```Swift
(lldb) po String(format: "%02x", buf[3])
"ffffffd2"

(lldb) po String(buf[3], radix: 16)
"-2e"
```

...負の値になってやがる...

※ちなみにC++ではこの後取り出したバッファを

```C++
sprintf(buf2, "%02x%02x%02x%02x", (unsigned char)buf[0], (unsigned char)buf[1], (unsigned char)buf[2], (unsigned char)buf[3])
```

で16進数8桁の文字列に結合した後、10進数に変換してる模様

## どうしてこうなった

C++(というかC)ではchar型が用意されてるけど、
SwiftではCのchar型に対応するものとして、こんな感じで定義してる

```Swift
public typealias CChar = Int8
```

つまり、数値として認識されちゃってると
Cのchar型も実態は符号付きintのはずなんだけどこちらは何故か正の値で表示されてる(Cにそこまで詳しく無いので見当違いの事言ってたら誰か教えてください...)

# やりたいこと

```Swift
(lldb) po String(format: "%02x", buf[3])
"ffffffd2"
```

この"ffffffd2"(-46)を何とかして"d2"(210)に変換する必要があった
けどCUnsignedChar(-46)ってやると落ちるしキャストも

```Swift
(lldb) po CChar(-46) as? CUnsignedChar
nil
```

こうなるし

...

~~あれ、ビット反転して255から引いてやれば終わりじゃね( ﾟдﾟ)~~

## 生まれたコード

```Swift
    /** バッファから自然数値を取り出す */
    static func readNInt(from buf: [CChar], size: Int = 4) -> Int {
        var hex = ""
        for i in 0 ..< size {
            hex += String(format: "%02x", unsignedChar(from: buf[i]))
        }
        return Int(hex, radix: 16) ?? 0
    }
    
    /** 符号付き整数型を符号無し整数型に変換する */
    private static func unsignedChar(from char: CChar) -> CUnsignedChar {
        if char >= 0 {
            return CUnsignedChar(char)
        }
        
        return CUnsignedChar.max - CUnsignedChar(~char)
    }
```

~~元のコードを踏襲してるのでもうちょっとスマートにはできるはずだけど一旦これで解決~~

### 11/10 追記

コメントでbitPatternイニシャライザなるものがあると教えて頂いたので早速活用

```Swift
    /** バッファから自然数値を取り出す */
    static func readNInt(from buf: [CChar], size: Int = 4) -> Int {
        var hex = ""
        for i in 0 ..< size {
            hex += String(format: "%02x", CUnsignedChar(bitPattern: buf[i]))
        }
        return Int(hex, radix: 16) ?? 0
    }
```

うん、スッキリ( 'ω')

## つまり(2の補数とか基礎的な話になるので分かる人はすっ飛ばしてね)

-46を符号付き8ビット整数に置き換えると
11010010
(負値の計算方法は[この辺](http://www.geocities.jp/zaqzaqpa/2sinsuu8.htm)参考にしてね)

で、これが符号無し8ビット整数とみなして10進数に変換してみると
210

なんと、"d2"になるのである( ﾟдﾟ)

Swiftでは直接キャストができないので、まず-46をビット反転させて
CUnsignedCharで符号無しに変換してやる
それをCUnsignedChar.max(つまり255)から引くことで
同じ2進数の値を取り出す事ができるという寸法だ

# というか

数値を取り出す時に128を超える値が入ってそうな場合に注意してねって話なんだろうね(´ω`)

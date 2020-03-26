# 概要
キーを指定するだけでサクッとソートしてくれるようなextensionを作ってみた


# 実装
```swift:Sequence+KeySort.swift
extension Sequence {
    public func sorted<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        sorted { $0[keyPath: keyPath] < $1[keyPath: keyPath] }
    }

    public func sorted<T: Comparable>(by keyPath: KeyPath<Element, T?>) -> [Element] {
        sorted {
            guard let l = $0[keyPath: keyPath],
                let r = $1[keyPath: keyPath] else { return false }
            return l < r
        }
    }
}
```

# Usage
```swift
struct Person: CustomStringConvertible {
    var id: Int
    var name: String

    var description: String { name }
}

let people: [Person] = [
    .init(id: 3, name: "Bob"),
    .init(id: 1, name: "Emma"),
    .init(id: 4, name: "Amelia"),
    .init(id: 2, name: "George"),
]

print(people.sorted(by: \.id))   // -> ["Emma", "George", "Bob", "Amelia"]
print(people.sorted(by: \.name)) // -> ["Amelia", "Bob", "Emma", "George"]
```

R.swiftはSwiftUIに直接対応していません。
ですが、extensionを生やすことで間接的に対応させることができそうです。

※随時更新していきます

# String編
```swift:Rswift+SwiftUI.swift
extension Text {
    init(_ resource: StringResourceType) {
        self.init(.init(resource.key), tableName: resource.tableName, bundle: resource.bundle)
    }
}
```

```swift:ContentView.swift
Text(R.string.localizable.hoge)
```

# Image編

```swift:Rswift+SwiftUI.swift
extension Image {
    init(_ resource: ImageResourceType) {
        self.init(resource.name, bundle: resource.bundle)
    }
}
```

```swift:ContentView.swift
Image(R.image.ic_launcher)
```

# NavigationBar編
```swift:Rswift+SwiftUI.swift
extension View {
    @available(OSX, unavailable)
    public func navigationBarTitle(_ resource: StringResourceType) -> some View {
        navigationBarTitle(Text(resource))
    }

    @available(OSX, unavailable)
    @available(tvOS, unavailable)
    @available(watchOS, unavailable)
    public func navigationBarTitle(_ resource: StringResourceType,
                                   displayMode: NavigationBarItem.TitleDisplayMode) -> some View {
        navigationBarTitle(Text(resource), displayMode: displayMode)
    }
}
```

```swift:ContentView.swift
VStack {
    ~
}
.navigationBarTitle(R.string.localizable.title, displayMode: .inline)
```

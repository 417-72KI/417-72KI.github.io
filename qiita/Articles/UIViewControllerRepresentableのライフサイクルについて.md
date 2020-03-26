
ふと気になったのでPlayground作って試してみた。

作ったPlaygroundは[GitHubにも公開](https://github.com/417-72KI/SwiftUIPlayground)しているので良かったら見てみてください。

# ソース

```swift
import SwiftUI
import UIKit
import PlaygroundSupport

struct SampleView: UIViewControllerRepresentable {
    typealias UIViewControllerType = SampleUIViewController

    func makeUIViewController(context: UIViewControllerRepresentableContext<SampleView>) -> SampleUIViewController {
        print(#function)
        return .init()
    }

    func updateUIViewController(_ uiViewController: SampleUIViewController, context: UIViewControllerRepresentableContext<SampleView>) {
        print(#function)
    }

    func makeCoordinator() -> Coordinator {
        print(#function)
        return .init()
    }
}

extension SampleView {
    class Coordinator: NSObject {
        override init() {
            super.init()
            print(self, #function)
        }
        deinit {
            print(self, #function)
        }
    }
}

class SampleUIViewController: UIViewController {
    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        print(self, #function)
    }

    init() {
        super.init(nibName: nil, bundle: nil)
        print(self, #function)
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
        print(self, #function)
    }

    override func loadView() {
        print(self, #function)
        super.loadView()

        let label = UILabel()
        label.text = "Hello world!"

        label.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(label)
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    override func viewDidLoad() {
        print(self, #function)
        super.viewDidLoad()
    }

    override func viewWillAppear(_ animated: Bool) {
        print(self, #function)
        super.viewWillAppear(animated)
    }

    override func viewWillLayoutSubviews() {
        print(self, #function)
        super.viewWillLayoutSubviews()
    }

    override func viewDidLayoutSubviews() {
        print(self, #function)
        super.viewDidLayoutSubviews()
    }

    override func viewDidAppear(_ animated: Bool) {
        print(self, #function)
        super.viewDidAppear(animated)
    }

    override func viewWillDisappear(_ animated: Bool) {
        print(self, #function)
        super.viewWillDisappear(animated)
    }

    override func viewDidDisappear(_ animated: Bool) {
        print(self, #function)
        super.viewDidDisappear(animated)
    }

    deinit {
        print(self, #function)
    }
}

do {
    let view = SampleView()
    print()
    PlaygroundPage.current.setLiveView(view)
    print()
}

let semaphore = DispatchSemaphore(value: 0)
DispatchQueue.global().async {
    sleep(5)
    semaphore.signal()
}
semaphore.wait()
PlaygroundPage.current.liveView = nil
```

# 結果

```
makeCoordinator()
<_TtCV14__lldb_expr_9010SampleView11Coordinator: 0x60000323fa90> init()
makeUIViewController(context:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> init()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> loadView()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewDidLoad()
updateUIViewController(_:context:)
updateUIViewController(_:context:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewWillAppear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewWillLayoutSubviews()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewDidLayoutSubviews()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewWillDisappear(_:)

makeCoordinator()
<_TtCV14__lldb_expr_9010SampleView11Coordinator: 0x60000323ff00> init()
makeUIViewController(context:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> init()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> loadView()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewWillAppear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewWillLayoutSubviews()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewDidLayoutSubviews()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewWillDisappear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewDidLoad()
updateUIViewController(_:context:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewWillLayoutSubviews()
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewDidLayoutSubviews()

<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewDidDisappear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> viewDidAppear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewDidDisappear(_:)
updateUIViewController(_:context:)
<_TtCV14__lldb_expr_9010SampleView11Coordinator: 0x60000323fa90> deinit
<__lldb_expr_90.SampleUIViewController: 0x7f88daee0370> deinit
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewDidAppear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewWillDisappear(_:)
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> viewDidDisappear(_:)
<_TtCV14__lldb_expr_9010SampleView11Coordinator: 0x60000323ff00> deinit
<__lldb_expr_90.SampleUIViewController: 0x7f88daee4c30> deinit
```

Playgroundでやったからなのかもしれないけど今までのライフサイクルと違う呼ばれ方をしてるのでほーんという感じ😇
気になったのはstructの生成時とliveViewへのセット時で`makeUIViewController()`が2回呼ばれていたこと。
画面によっては切り替えの度にmakeUIViewControllerが呼ばれてVCが生成されまくるんじゃなかろうか🤔

# 参考
[UIViewControllerのライフサイクル](https://qiita.com/motokiee/items/0ca628b4cc74c8c5599d)

※随時更新

# プラグインを入れる
## [Smart split into lines](https://marketplace.visualstudio.com/items?itemName=JulioGold.vscode-smart-split-into-lines)
複数行選択したらそれを複数カーソルに変えてくれる。(Sublime Textの `cmd+shift+l` に当たる機能)

# ユーザー設定(settings.json)
```json
{
    "java.home": "/Library/Java/JavaVirtualMachines/jdk-9.0.4.jdk/Contents/Home",
    "python.pythonPath": "python3",
    "explorer.confirmDragAndDrop": false,
    "window.zoomLevel": 0,
    "git.confirmSync": false,
    "git.autofetch": false,
    "explorer.confirmDelete": false,
    "editor.suggestSelection": "first",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue"
}
```

# キーバインドの設定(keybindings.json)
```json
[
    {
        "key": "shift+cmd+l",
        "command": "smart.splitIntoLines",
        "when": "editorTextFocus"
    },
    {
        "key": "ctrl+cmd+u",
        "command": "editor.action.transformToUppercase"
    },
    {
        "key": "ctrl+cmd+l",
        "command": "editor.action.transformToLowercase"
    }
]
```

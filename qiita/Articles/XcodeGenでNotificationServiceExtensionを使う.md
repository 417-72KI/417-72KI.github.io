# `targets`にこんな感じで追加する

```yaml:project.yml  
  NotificationService:
    type: app-extension
    platform: iOS    
    sources: NotificationService
    settings:
      base:
        ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES: ${inherited}
        PRODUCT_NAME: NotificationService
        PRODUCT_BUNDLE_IDENTIFIER: {NotificationService用のBundle Identifier}
        INFOPLIST_FILE: NotificationService/info.plist
        CODE_SIGN_STYLE: Manual
        TARGETED_DEVICE_FAMILY: 1
        DEVELOPMENT_TEAM: {アプリに合わせる}
      configs:
        debug: 
          CODE_SIGN_IDENTITY: iPhone Developer
          PROVISIONING_PROFILE_SPECIFIER: {Debugビルド用のProvisioning Profile}
        release: 
          CODE_SIGN_IDENTITY: iPhone Distribution
          PROVISIONING_PROFILE_SPECIFIER: {Releaseビルド用のProvisioning Profile}
```

# アプリターゲットの`dependencies`に以下を追加

```yaml:project.yml
- target: NotificationService
  codeSign: false
  embed: true
```

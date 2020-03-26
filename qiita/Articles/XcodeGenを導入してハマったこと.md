何かあったら追記していくかも

# ブランチを切り替える際にファイルが追加/削除されていてもプロジェクトに反映されない
まあ*project.pbx*が管理されないので当然である(
とはいえ`git checkout`を走らせる度に毎回XcodeGenと`pod install`走らせるのめんどい

そこで[Git hooks](https://git-scm.com/book/ja/v1/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%83%95%E3%83%83%E3%82%AF)なる仕組みを発見したのでこれを使うことにしました

```sh:.git/hooks/post-checkout
#!/bin/sh

# エラーになった時点で終了
set -e

echo "Post Checkout"
/usr/local/bin/mint run xcodegen
/usr/local/bin/pod install
```

これで`git checkout`の度に自動でプロジェクト構成を更新してくれるようになりました。

ちなみになぜ`mint`と`pod`をフルパスで書いてるかって言うと、GUIツールでブランチを切り替える時にPATHがターミナル実行時と違うために、`/usr/local/bin`がPATHとして認識されなかったためです。
(SourceTreeも使うのでその度にエラー吐かれて鬱陶しい)

```sh
$ gout
Switched to branch 'hoge'
Post Checkout
🌱  XcodeGen 2.1.0 already installed
🌱  Running xcodegen 2.1.0...
(中略)
Analyzing dependencies
Downloading dependencies
(中略)
Generating Pods project
Integrating client project
Sending stats
Pod installation complete! There are 11 dependencies from the Podfile and 14 total pods installed.
```

## 2020/02/13 追記
本当に毎回走られると流石に鬱陶しくなってくるので、`git checkout`時にcheckout前後の差分に応じて走らせるようにしてみました。

```sh:post-checkout
#!/bin/zsh

# エラーになった時点で終了
set -e

# GUIツール対策
PATH=$HOME/.rbenv/shims:/usr/local/bin:$PATH

PREV_COMMIT=$1
POST_COMMIT=$2

DIFF=$(git diff $PREV_COMMIT..$POST_COMMIT --name-status --diff-filter=ADR | grep -e project.yml -e .swift)
if [[ $DIFF != "" ]]; then
    while read line
    do
        echo "$line"
    done <<END
    $DIFF
END
    mint run xcodegen
    pod install
fi
```

# CI上でR.swift等の自動生成系がコケる
*project.pbx*が(ry

<details><summary>最初のアプローチ</summary>
<div>
ビルドスクリプト自体は<details><summary>*project.yml*</summary>
<div>

```yml:project.yml
    preBuildScripts:
      - name: Lint
        script: |
                if which swiftlint >/dev/null; then
                swiftlint
                else
                echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
                fi
      - name: BuildConfig.swift
        script: |
                if [ "${CONFIGURATION}" = 'Release' ]; then
                ENVIRONMENT='production'
                else
                ENVIRONMENT='staging'
                fi
                ${PODS_ROOT}/BuildConfig.swift/buildconfigswift -o ${SRCROOT}/Libs/BuildConfig_swift -e $ENVIRONMENT ${SRCROOT}/Resources/Config
        inputFiles:
          - $(TEMP_DIR)/buildconfigswift-lastrun
        outputFiles:
          - $(SRCROOT)/Libs/BuildConfig_swift/BuildConfig.generated.swift
          - $(SRCROOT)/Libs/BuildConfig_swift/BuildConfig.plist
      - name: R.swift
        script: ${PODS_ROOT}/R.swift/rswift generate ${SRCROOT}/Libs/R_swift/R.generated.swift
        inputFiles:
          - $(TEMP_DIR)/rswift-lastrun
        outputFiles:
          - $(SRCROOT)/Libs/R_swift/R.generated.swift
```

</div>
</details>で定義していたので、

`outputFiles`に合わせて初期化用のlaneを作り、そこで自動生成ファイルを先に作るようにしました

```ruby:Fastfile
  desc "Initialize project"
  lane :init_project do
    lib_dir = 'Libs'
    sh("cd .. && mkdir -p #{lib_dir}/R_swift #{lib_dir}/BuildConfig_swift && touch #{lib_dir}/R_swift/R.generated.swift #{lib_dir}/BuildConfig_swift/BuildConfig.generated.swift #{lib_dir}/BuildConfig_swift/BuildConfig.plist")

    carthage(
      cache_builds: true,
      platform: 'iOS'
    )
    sh("cd .. && mint run xcodegen")
    cocoapods(repo_update: false)
  end
```

これでCI上でも自動生成系のファイルがXcodeGenの実行よりも先に生成されるので*project.pbxproj*に取り込まれるようになりました。

また、新しいメンバーが入ってきた際の開発環境構築でも同じ問題に遭遇するので、fastlaneに疎い人でも構築しやすいように`Makefile`を用意しました

```make:Makefile
init:
    bundle install --quiet
    bundle exec fastlane init_project
```

</div></details>

## 2019/5/15追記

@yosshi4486さんから[ブログ記事](http://takkkun.hatenablog.com)を紹介してもらいました。
*project.yml*で完結できるのでこちらの方が良さそうですね😃
最終的に`sources`に以下のように追加することで解決しました。

```yaml:project.yml
- path: Libs/R_swift/R.generated.swift
  optional: true
  createIntermediateGroups: true
- path: Libs/BuildConfig_swift/BuildConfig.generated.swift
  optional: true
  createIntermediateGroups: true
- path: Libs/BuildConfig_swift/BuildConfig.plist
  optional: true
  createIntermediateGroups: true
```

`createIntermediateGroups`をtrueにしないと`Libs`グループで括ってくれないので注意。

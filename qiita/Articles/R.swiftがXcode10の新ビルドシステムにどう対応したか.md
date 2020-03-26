~~つい先日**R.swift**が`5.0.0.rc.1`になりました~~
~~https://github.com/mac-cain13/R.swift/releases/tag/v5.0.0.rc.1~~

~~まだプレリリース版ではありますが、これでXcode10対応がほぼできたと見れそうです。~~

12/18付で`5.0.0`が正式版としてリリースされたようです。やったね！
https://github.com/mac-cain13/R.swift/releases/tag/v5.0.0

# どんな問題が起きていたか
詳しいやり取りは[このIssue](https://github.com/mac-cain13/R.swift/issues/456)でされていますが、Xcode10から標準になった新しいビルドシステムを使うとR.swiftが上手く動かないという問題がありました。

新しいビルドシステムではビルド時間短縮のために各Build Phaseを並列処理しているようなのですが、それによってR.swiftの実行を待たずにコンパイルに入ろうとするため、リソースの更新が上手くされなかったり、CIサービス上でビルドエラーになったりしていました。

# 解決策1
R.swift実行用Build Phaseの**Output Files**に*R.generated.swift*へのパスを追加すると、
このBuild Phaseを待って次のBuild Phaseを実行するようになりました。

ところが、Output Filesにファイルが指定されると、どういうわけか２回目以降R.swiftが実行されなくなりました。
恐らくInput Filesが空なので変更が無いとみなされてスキップされるようです。

[robinkunde氏によって提案されていた](https://github.com/mac-cain13/R.swift/issues/456#issuecomment-426344064)のはBuild Phaseに`date > 適当なファイル`を追加し、
そのファイルをInput Filesに指定してやるという方法でした。

Input Filesが指定されている場合、そのファイルに変更があればそのBuild Phaseを実行するようなので、
Build Phaseが実行される度に変更されるファイルを用意して、そのファイルをInput Filesに指定すると良いみたいです。

# 解決策2
[tomlokhorst氏によると](https://github.com/mac-cain13/R.swift/issues/456#issuecomment-438734079)、Appleのテクニカルサポートから提案された方法として`Edit Scheme -> Build Pre-action`で予めファイルを作成するようにする方法を提示しました。

ビルドの前に実行されることが保証されるので、少なくともCI等でR.generated.swiftが無い状態からビルドを走らせるとコケるといったことは防げます。(けどこれBuild Phaseがスキップされる問題は解消できるんだろうか🤔)

# 結論
最終的にR.swiftでは解決策1を採用し、日付を`$(TEMP_DIR)`に書き出す処理を`rswift generate`の処理に入れ込むことで決着したようです。
これに伴い、Input Files及びOutput Filesに対してバリデーションをかけるようになりました。

これからR.swiftを導入しようとしている方や、R.swiftを4.x系から更新しようとしている方はご注意ください。

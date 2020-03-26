
Mac Appの開発でインストーラ(~.pkgってやつ)を作ってたんだけど、`pkgbuild`でインストール前後にシェルスクリプトを差し込めるそうなので試してみた。

# 注意
開発の都合で古いバージョンを使ってやってます。
- Xcode 6.1
- MacOS SDK 10.7

# 手順

## 1. スクリプトファイルを作る

- preinstall → インストール前に実行するシェルスクリプト
- postinstall → インストール後に実行するシェルスクリプト
	
今回はこんな感じのフォルダ構成に(プロジェクト名をHogeとする)
	
```
├ Hoge.xcodeproj
├ Hoge
└ InstallScripts
  ├ preinstall
  └ postinstall
```
	
※この時、preinstallとpostinstallのパーミッションに気をつけること
自分は実行権限が付いていなくてパッケージング完了後にインストールを試したら失敗しましたｗ
多分755が無難？

## 2. アーカイブを作成する
この辺はiOSと同じ。
XcodeでProduct→Archiveを選んでアーカイブ作成

## 3. appファイルにエクスポートする
ここも大体iOSと同じ。
違うのはエクスポートの選択肢
![Kobito.8afL1z.png](https://qiita-image-store.s3.amazonaws.com/0/135304/f19536a2-4758-7241-7fc3-3a62bdac1dbc.png "Kobito.8afL1z.png")
ここでは一番上の`Export a Developer ID-signed Application`を選択

## 4. `pkgbuild`を実行する

```
pkgbuild \
--component Hoge.app \
--identifier com.hoge.Hoge \
--install-location /Applications \
--version "Hogeのバージョン" \
--scripts /path/to/InstallScripts \
HogeInstaller.pkg
```

増えるのは--scriptsのオプション。
ここで指定したパスの中にあるpreinstallとpostinstallを取り込む模様

## 5. pkgファイルの出来上がり
あとはこのpkgを実行してインストールが完了すれば無事スクリプトが実行されてるはず

## ちなみに
pkgからのインストールで失敗した場合、下記のコマンドでダンプが取れるとか取れないとか

`sudo installer -dumplog -pkg HogeInstaller.pkg -target /`

# 考察
これでインストール時にApplication Support配下のファイルを見に行って特定のファイルを消すとかリネームするとかできそう( 'ω')

しっかしiOSと違ってOSXだと日本語どころか英語の資料も少なくてつらたん...

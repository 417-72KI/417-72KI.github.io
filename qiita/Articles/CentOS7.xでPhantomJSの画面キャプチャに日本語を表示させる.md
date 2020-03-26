
# 始めに

CentOSでPhantomJSを使って画面キャプチャを撮ったら日本語が表示されなくて、調べてみたら先人の記事があった。

[CentOSからPhantomJSを実行すると画面キャプチャの日本語が表示されなくなる問題 - Qiita](http://qiita.com/n-takahashi/items/3dd810ebdf236fce6f13)

ところが、この途中で

```
$ sudo yum groupinstall "Japanese Support"
読み込んだプラグイン:fastestmirror
There is no installed groups file.
Maybe run: yum groups mark convert (see man yum)
Loading mirror speeds from cached hostfile
 * base: ftp.iij.ad.jp
 * epel: ftp.riken.jp
 * extras: ftp.iij.ad.jp
 * updates: ftp.iij.ad.jp
Warning: Group japanese-support does not have any packages to install.
Maybe run: yum groups mark install (see man yum)
インストールまたは更新に利用できるいくつかの要求されたグループにパッケージがありません
```
となって日本語フォントがインストールされず詰まった...

# どうやら

日本語フォントのインストール方法がCentOS7.xで変わったらしい。
そのコマンドがこちら
`yum install ibus-kkc vlgothic-*`

この後
> フォント設定ファイルを修正

して、
> フォントキャッシュ更新

したら無事PhantomJSのキャプチャで日本語が表示されるようになった。

# ところで
試しに
> フォント設定ファイルを修正

の項目を元にもどして再度フォントキャッシュを更新してもう一度キャプチャを撮ってみたところ問題なく日本語が表示されたので、
もしかしたら
> フォント設定ファイルを修正

の項目はCentOS7.xではすっ飛ばしても良いのかも。

# 参考
[CentOSからPhantomJSを実行すると画面キャプチャの日本語が表示されなくなる問題 - Qiita](http://qiita.com/n-takahashi/items/3dd810ebdf236fce6f13)
[CentOS7の日本語化（日本語環境で利用する）- newsearch.okinawaの技術ブログ](http://vps-okinawa-blog.net/?p=97)

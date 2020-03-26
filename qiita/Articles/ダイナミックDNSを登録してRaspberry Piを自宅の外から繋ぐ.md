
家に帰らないとラズパイと戯れることができないので家の外からでもアクセスできるようにダイナミックDNSを登録してみた時のメモ書き

# ポートマッピングの設定
ルーターによって設定方法が違うので割愛

# ダイナミックDNSサービスに登録
無料のダイナミックDNSサービスは色々あるけど会社の先輩に教えてもらった[No-IP](https://my.noip.com/)というサービスを試してみることに
英語だけど直感的に登録できるので説明は割愛(おい

# ddclientをインストール
[DiCE for Linux](http://www.hi-ho.ne.jp/yoshihiro_e/dice/linux.html)は古すぎるのかラズパイがバイナリを読んでくれなかった
その代わりddclientというモジュールが使えるそうなので、[ここ](http://denshikousaku.net/raspberry-pi-domain-and-dynamic-dns)を参考にインストール

参考サイトによるとMyDNSはddclientが使えないらしいけど、No-IP使っているのでddclientが使えるか不明。ということで一旦様子見( 'ω')

# メモ
- auひかりはグローバルIPのリースタイミングが1日間隔になっているそうで、環境によっては事実上の固定IPになる場合もあるらしい

*** 

10/7 追記

# DUCをインストール

しばらくddclientの様子見てみたけどどうも動いてないようなのでNo-IP公式から用意されているDynamic Update Client(DUC)を試してみる。

1. 適当なディレクトリにパッケージをダウンロード

	```
	wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz
	```

2. パッケージを解凍

	```
	tar xf noip-duc-linux.tar.gz
	```

3. 解凍してできたディレクトリに移動してインストール開始

	```
	cd noip-2.1.9-1/
	sudo make install
	```

4. ログイン設定

	```
	pi@raspberrypi:~/ddns_client/noip-2.1.9-1$ sudo make install
	if [ ! -d /usr/local/bin ]; then mkdir -p /usr/local/bin;fi
	if [ ! -d /usr/local/etc ]; then mkdir -p /usr/local/etc;fi
	cp noip2 /usr/local/bin/noip2
	/usr/local/bin/noip2 -C -c /tmp/no-ip2.conf

	Auto configuration for Linux client of no-ip.com.

	Please enter the login/email string for no-ip.com  (ログインIDを入力)
	Please enter the password for user 'ログインID'  (パスワードを入力)
	```

5. その他の設定
	ここはなんとなく訳してるのであやふやです(

	```
	Only one host [(登録してるホスト)] is registered to this account.
	It will be used.
	Please enter an update interval:[30]  (更新間隔を設定？)
	Do you wish to run something at successful update?[N] (y/N)  (更新後に何かスクリプトを走らせるならy)
	Please enter the script/program name  (更新後に走らせたいスクリプトのパスを入れるっぽい？)
	```

# さて
これでちゃんと動いてくれるといいけど。。。

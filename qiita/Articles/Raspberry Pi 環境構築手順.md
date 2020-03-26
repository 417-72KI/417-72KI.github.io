RaspberryPiを買ってみたのでとりあえずMacからSSHで繋げるようにするとこまでのメモ書き

#買ったもの
[kksmart Raspberry Pi 3 Model B ラズベリーパイ 3 モデル B コンプリートスターターキット 16GB (class 10)](https://www.amazon.co.jp/kksmart-Raspberry-Model-%E3%83%A9%E3%82%BA%E3%83%99%E3%83%AA%E3%83%BC%E3%83%91%E3%82%A4-%E3%82%B3%E3%83%B3%E3%83%97%E3%83%AA%E3%83%BC%E3%83%88%E3%82%B9%E3%82%BF%E3%83%BC%E3%82%BF%E3%83%BC%E3%82%AD%E3%83%83%E3%83%88/dp/B01G8EZSCQ?ie=UTF8&ref_=pe_1807052_198774502_tnp_email_dp_1)
内容物は以下
- Raspberry Pi 3 Model B 本体
- Raspberry Pi2B/3B専用のABSケース（クリア）
- モニター用HDMIカーブル（1M）
- 16GB microSDカード(class 10)
- ヒートシンク
- USBカードリーダー
- 5V 2.5A RPI3専用電源アダプタ1.15M（PSE対応）

#構築時に必要なもの
- ディスプレイ(HDMIで繋ぐか変換アダプタを使ってVGIとかに変換する)
- USBキーボード
- USBマウス

#インストール
1. microSDを初期化
[SDカードフォーマッター](https://www.sdcard.org/jp/downloads/formatter_4/)とやらを使うのが定番らしい
Mac版もあったのでそれで

2. NOOBSをダウンロード
  https://downloads.raspberrypi.org/NOOBS_latest から
  1.07GBあるので気長に待つよ
>NOOBSとは
>ラズパイ用のOSインストーラー
>Raspbianの他にも色々なOSが入ってる

3. NOOBSを解凍
ダウンロードしたNOOBS_vx_x_x.zip(この記事を書いた時点ではNOOBS_v1_9_2.zip)を解凍する
解凍ソフトによってはサイズの関係で解凍できないらしいけどMacの標準解凍ツールは優秀だった( 'ω')

4. NOOBSをSDカードにコピー
解凍したNOOBSを初期化したSDカードのルートにコピーする
※ コピーする際は解凍してできたフォルダじゃなくてファイル群をルートに入れること

5. 起動
キーボードとかマウスとかディスプレイとか繋いだらmicroUSB端子にACアダプタを繋ぐ
余談だけどラズパイの面白いとこって電源繋いだら勝手に起動してくれるところよね( 'ω')
つまりシャットダウンしたらmicroUSBを抜き差ししなきゃいけないわけだけど(

6. インストールするOSを選択
NOOBSがちゃんとコピーされてれば画面にOS選択画面が表示されるはず
※ 間違えてフォルダごとコピーしたらモニターに信号が送られず一旦切る羽目になったよ(
マウスでRaspbianを選択してウィンドウ左上のインストールボタンをクリック
この時に下部で言語も選択できるみたいなので好みで日本語にしておく

7. のんびり待つ(
意外に時間がかかる
暇つぶしにポケモンGoでもどうぞ(
インジケーターが100%になったら「インストールが完了しました」とダイアログが出て、
OKをクリックすると再起動して自動ログインまでしてくれる

#起動オプションを設定
GUIを使う予定が無いのでコンソール起動にする
1. コンソールを起動
上のツールバーにそれっぽいアイコンがあるのでそれをクリックすればコンソールが開く
2. Raspberry Pi Software Configuration Toolを起動
`sudo raspi-config`と打ち込むと画面が切り替わる
3. Boot Optionsを選択
4. B1 Consoleを選択
※ B4 Desktop Autologinが現在の設定なので、GUIで起動したい場合はそのまま
5. Finishを選択
再起動を促されるので再起動する

#SSHログインできるようにする
SSH自体は標準で入ってるっぽいので固定IPさえ設定してしまえば良さそう

1. dhcpcpファイルを開く
~~vimが入っていないため、~~ラズパイ標準(?)のnanoというエディタを使う
***
**2016/8/1追記**
viは入ってたけどmacからsshで繋いで動かしてると矢印キーで変な挙動起こされたので使わない方が良い
nanoも悪くは無いけどこだわるならvim落としてきた方が良いかも
***
`sudo nano /etc/dhcpcd.conf`

2. eth0設定を末尾に追記する

```bash:dhcpcd.conf
interface eth0
static ip_address=192.168.0.250/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
```
`Ctrl+O`で上書き保存
`Ctrl+X`で終了
3. ネットワーク設定を読み込む
`sudo /etc/init.d/dhcpcd reload`
4. 確認
Macから
`ping 192.168.0.250`
を打ってパケットが返ってくれば成功
5. ログインしてみる
Macから
`ssh pi@192.168.0.250`
パスワードは`raspberry`
ログインできれば成功


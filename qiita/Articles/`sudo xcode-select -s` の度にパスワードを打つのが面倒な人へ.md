`xcode-select` の実行時のみ `sudo` のパスワード入力をスキップできるように `visudo` で1行付け加えるだけ

```sh:visudo
# root and users in group wheel can run anything on any machine as any user
root            ALL = (ALL) ALL
%admin          ALL = (ALL) ALL

{input-your-user-name}           ALL = NOPASSWD: /usr/bin/xcode-select # この1行を追加

```

# 注意
特にいじっていない初期状態のファイルであれば、
`# User specification`というブロックがあるが、ここではなくその下の
`# root and users ~`のブロックの一番下に書くこと。
(`# root and users ~`直下の設定で上書きされるっぽい？)

# 参考
[sudo のパスワードを入力なしで使うには](https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7)
[MacOSで「sudo」コマンドをパスワードなしにする方法](https://ex1.m-yabe.com/archives/2721)

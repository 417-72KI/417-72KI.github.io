Macを買い替えた時や業務で新しいMacを支給された時などに毎回設定するの面倒なのでターミナルで走らせる系の作業をまとめてみました。

必要な作業を思い出したら随時スクリプトを追記していきます。

# `.bash_profile` を作成

```sh
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

# エイリアスを登録する

```sh
cat << EOS >> ~/.bashrc
alias ls='ls -G'
alias ll='ls -lha'
alias rmdsstore="find . -name '*.DS_Store' -type f -ls -delete"
alias relogin='exec $SHELL -l'
alias merged_branch='git branch --merged | grep -vE '\''^\*|master$'\'''
alias rmmerged_branch='merged_branch | xargs -I % git branch -d %'
alias rmderived='rm -rf ~/Library/Developer/Xcode/DerivedData/*'
alias gb='git branch'
alias gba='git branch -a'
alias gf='git fetch -p'
alias gp='git pull --rebase -p'
alias grb='SKIP_POST_CHECKOUT=1 git rebase $(gba | grep -v "HEAD" | peco | tr -d '"'"' '"'"' | tr -d '"'"'*'"'"')'
alias grm='SKIP_POST_CHECKOUT=1 gf && git rebase origin/master'
alias gp='git pull --rebase -p'
alias grc='git rebase --continue'
alias gout='git checkout \$(git branch | peco | tr -d '"'"' '"'"' | tr -d '"'"'*'"'"')'
alias glook='ghq look \$(ghq list | peco)'
EOS
```

# ネットワークドライブ(NASとか)で `.DS_Store` を作成しないようにする
```sh
defaults write com.apple.desktopservices DSDontWriteNetworkStores True
killall Finder
```

# 隠しファイルを表示するようにする

```sh
defaults write com.apple.finder AppleShowAllFiles TRUE
killall Finder
```

# Homebrewインストール

```sh
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

# rbenvインストール

```sh
brew install rbenv rbenv-communal-gems
```

※.bashrcに以下を追記

```sh:.bashrc
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

# rbenvでRubyを最新版に

```sh
rbenv install 2.6.0
rbenv global 2.6.0
```

# Node.jsインストール

```sh

brew install nodebrew
mkdir -p ~/.nodebrew/src
nodebrew install-binary latest
echo 'export PATH=$PATH:~/.nodebrew/current/bin' >> ~/.bashrc
```

# Dockerインストール
```sh
brew cask install docker
open /Applications/Docker.app

curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

# ghq & pecoインストール
```sh
brew install ghq
brew install peco
```

# 最後に
↑を全部まとめてやってくれるスクリプトを[GitHub](https://github.com/417-72KI/MacUtilties/blob/master/init_mac.sh)にまとめてみました。

# 参考
- [シェルを再起動させる簡単な方法](https://qiita.com/yusabana/items/c4de582c6f85a42817d8)
- [【Mac】隠しファイル・隠しフォルダを表示する方法](https://qiita.com/TsukasaHasegawa/items/fa8e783a556dc1a08f51)
- [.DS_Storeをネットワークドライブ上で作成しないようにする](http://ftvoid.com/blog/post/642)
- [【2018年版】macにrbenvを入れてrubyを管理できるようにしちゃう](https://qiita.com/Alex_mht_code/items/d2db2eba17830e36a5f1)
- [rbenvでRubyの最新安定版をインストールするワンライナー](https://mawatari.jp/archives/install-latest-stable-version-of-ruby-using-rbenv)


新しいプロジェクトに入る度に開発環境が変わるので、その度にApacheサーバやら何やらを消しては入れ直しってやるのがいい加減面倒になってきたのでDockerの勉強がてら開発環境構築を自動化してみようという計画

Docker単体での運用でもいいけどDocker for Macは遅いっていう話を聞くし、Windowsだとそもそも10以外門前払いだし...
じゃあVM挟んで運用すればいいよね！( ﾟдﾟ)
けどVMの構築が手間...
そうだ、Vagrantを使おう( ﾟдﾟ)

# その前に

## Vagrantって？

仮想マシン(VM)の構築を自動化してくれるツール。
VirtualBoxと組み合わせて使う事が多い。

## Dockerって？

Linuxコンテナという機能を良しなに使いやすくしたツール。
コンテナのお話とかはちょっと説明が難しいので[この辺](http://www.atmarkit.co.jp/ait/articles/1701/30/news037.html)とか[この辺](https://cloudpack.media/30811)を参考にするとよろし

# 前提

- 既にある程度開発が進んでるLaravelプロジェクト
- プロジェクトは既にGit管理されている
- 既存の開発環境構築手順があるのでなるべくそれに合わせる
- DBは別途検証用のマシンに入れてるので今回の構成には組み込まない

## 実行環境
- OS: MacOS 10.12 Sierra
- VirtualBox: 5.0.32
- Vagrant: 1.9.1
- Docker: 17.06.0-ce, build 02c1d87

## 使用するVM
CentOS7

## アプリケーション
- Webサーバ: Apache
- PHP: 7.0.*
- Laravel: 5.3.29

# 実際にやってみた

## 構成

```
path/to/project
├── app/
├── Vagrantfile
├── docker-compose.yml
├── php/
│   ├── Dockerfile
│   ├── mod_rewrite.conf
│   ├── app.conf
│   └── php.ini
└── provision_init.sh
```

- app: アプリケーションが格納されたフォルダ。これを展開するとLaravelのディレクトリ構成になってる
- Vagrantfile: Vagrantの設定ファイル
- docker-compose.yml: Docker Composeの設定ファイル
- php: Docker Composeで使うフォルダ
	- Dockerfile: Dockerイメージを作成するための設定ファイル
	- mod_rewrite.conf, app.conf: httpd.confのIncludeとかIncludeOptionalで読み込ませるファイル。httpd.confを直接いじるよりも手間がかからない。
	- php.ini: PHPの設定ファイル
- provision_init.sh: Vagrantでの仮想マシン初期構築時に走るスクリプト。この中でDockerをインストールする。

## Vagrantの設定

```Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "app.dev"

  if ENV['VAGRANT_BRIDGE']
      interfaces  = %x(VBoxManage list bridgedifs)
      re          = /Name: +(.*#{ENV['VAGRANT_BRIDGE']}.*)/
      if interfaces =~ re
          config.vm.network :public_network, bridge: $1
      end
  else
      interfaces  = %x(VBoxManage list bridgedifs)
      if interfaces =~ /Name: *(.+)\n/
          config.vm.network :public_network, bridge: $1
      end
  end

  config.vm.synced_folder "./", "/vagrant", type: "virtualbox", mount_options: ['dmode=777','fmode=775']

   config.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
   end


  config.vm.provision "shell", :path => "provision_init.sh", :privileged => false
  config.vm.provision "shell", run: "always", inline: "systemctl restart network.service"
  config.vm.provision "shell", run: "always", inline: "service docker start"
end
```

```provision_init.sh
sudo yum update -y
sudo yum install wget -y
sudo curl -fsSL https://get.docker.com/ | sh
sudo usermod -aG docker vagrant
sudo sh -c "curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
sudo chmod +x /usr/local/bin/docker-compose
```

## Dockerの設定

```docker-compose.yml
version: '2'
services:
  php:
    build: ./php
    ports:
      - '80:80'
    volumes:
      - /vagrant/app:/app:rw

```

```Dockerfile
FROM centos:latest
RUN yum -y update \
&& yum -y install epel-release http://rpms.famillecollet.com/enterprise/remi-release-7.rpm \
&& yum -y install wget httpd git \
&& yum -y --enablerepo=remi-php70 install php php-mbstring php-xml php-pdo php-pgsql \
&& yum clean all

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY php.ini /etc/
COPY app.conf /etc/httpd/conf.d/
COPY mod_rewrite.conf /etc/httpd/conf.modules.d/

RUN mkdir /app
WORKDIR /app
VOLUME /app

RUN groupadd -g 1000 vagrant \
&& useradd -s /bin/bash -u 1000 -g vagrant vagrant \
&& usermod -aG vagrant apache

EXPOSE 80
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

```

```app.conf
<VirtualHost *:80>
    DocumentRoot "/app/public"
    <Directory "/app/public">
        AllowOverride All
        Require all granted
        EnableMMAP Off
        EnableSendfile Off
    </Directory>
</VirtualHost>
```

```mod_rewrite.conf
LoadModule rewrite_module modules/mod_rewrite.so
```

## 起動

```
$ cd path/to/project
$ vagrant up
$ vagrant ssh
# ここからVM内
$ cd /vagrant
$ docker-compose up -d
```

VM内で `ip a` と叩けばVMに割り当てられたIPアドレスが確認できるので、
そのIPアドレスをブラウザで直接開いてアプリケーションのトップ画面が表示されればクリア。

# 軽く解説

## Vagrantfileについて
`config.vm.network :public_network` って設定するとネットワークの割り当てにブリッジアダプタを使用するようになる。
ただそれだけだと `Vagrant up` の度に使用するネットワークアダプタを選ばないといけないので環境変数で設定したり良しなにやってくれるようにしてるのが5行目〜16行目。参考記事も参照されたし。


## Dockerfileについて
DockerHubに公開されてるphp-apacheイメージがDebianベースだったので今回の要件に合わず。
やむを得ずCentOSのイメージをベースにApacheとPHPを入れる形に。


## アプリケーションフォルダのマウント

```Vagrantfile
config.vm.synced_folder "./", "/vagrant", type: "virtualbox", mount_options: ['dmode=777','fmode=775']
```
```docker-compose.yml
    volumes:
      - /vagrant/app:/app:rw
```
```Dockerfile
RUN mkdir /app
WORKDIR /app
VOLUME /app
```

実マシンの `path/to/project` が Vagrantマシン内で `/vagrant` としてマウントされ、その中の `app` ディレクトリをDockerコンテナ内で `/app` としてマウントしている。
こうすることでソースコードの編集を実マシン上で行うことができ、デプロイ作業等をすることなくコンテナに変更が反映される。
但し、これだけだとコンテナ内のApacheアプリケーションがパーミッションエラーを起こす( `/app/storage` に書き込みができない)ので、


```Dockerfile
RUN groupadd -g 1000 vagrant \
&& useradd -s /bin/bash -u 1000 -g vagrant vagrant \
&& usermod -aG vagrant apache
```

でapacheユーザ(Apacheアプリケーションが使用するユーザ)をvagrantグループに登録してやると、apacheユーザに `/app/` 以下への書き込み権限が付与される。
(先のVagrantfileで `config.vm.synced_folder` に `mount_options` を設定しているのもこのため)

# まとめとか

今回は既にある程度の規模で開発が進んでるプロジェクトだったのでほぼ自己満で終わってるけど、今後新規開発の話があった時の環境構築のベースにしたい。(実はWindows上での検証ができてないのでそれが先...)
あと使うコンテナが1つだけで、ポートとかマウントの設定のためだけにDocker Composeを使ってるのは勿体無いので、他にもコンテナを乗っけて連携とかできればいいかな。
将来的には検証環境や本番環境のアプリケーションもDockerベースで運用できれば最the高。

# 参考
- [Vagrant+Dockerローカル用LAMP環境作成メモ](http://qiita.com/c_obara/items/b0d7ca5364ccaa9232ee)
- [Vagrant + docker内のコンテナとVMホスト間でファイルを共有する](http://qiita.com/h_demon/items/b77be877e9c517a074f9)
- [環境変数で Vagrant の bridge を指定できるようにする](http://qiita.com/elim/items/816f03c732e4b274d181)
- [vagrant上、共有ファイルのパーミッションを変更する](http://qiita.com/YusukeHigaki/items/ca36a5a3d27417e67ac7)


MySQLの公式イメージをベースに初期データを投入する **Dockerfile** を作って `docker-compose up` する度に初期DBができるようにしたものの、実際に動かす時に引っかかったのでメモ

# 何がしたかったかと何が起きたか
要は `docker-compose up` の度にDBを初期状態にしたかったんだけど、初期化用のSQLを変えた時に何故かそれが反映されない。
というか `docker-compose exec mysql bash` でコンテナに入った後MySQLにログインして操作した履歴が残るんですけどー！！

## Docker周りの設定ファイル

```docker-compose.yml
version: '3'
services:
  mysql:
    build: ./mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - '3306:3306'
```

```Dockerfile
FROM mysql

RUN apt-get clean && apt-get update && apt-get install -y locales
RUN locale-gen ja_JP.UTF-8
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:en
ENV LC_ALL ja_JP.UTF-8
RUN localedef -f UTF-8 -i ja_JP ja_JP.UTF-8

RUN { \
    echo '[mysqld]'; \
    echo 'character-set-server=utf8'; \
    echo 'collation-server=utf8_general_ci'; \
    echo ''; \
    echo '[client]'; \
    echo 'default-character-set=utf8'; \
    echo ''; \
    echo '[mysql]'; \
    echo 'default-character-set=utf8'; \
} > /etc/mysql/conf.d/charset.cnf

ADD ./login.sh /root/login.sh

ADD ./init_db.sql /docker-entrypoint-initdb.d/01_init_db.sql
ADD ./init_tables.sh /docker-entrypoint-initdb.d/02_init_tables.sh
```

# 原因

`docker-compose up mysql` でコンテナを起動すると途中でこんなログが出て来る

```
mysql_1   | /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/01_init_db.sql
```

じゃあ初期クエリは読まれてるのかってことで一旦 `Ctrl+C` でコンテナを止めて再度 `docker-compose up mysql` で起動...

( ﾟдﾟ)ん？

( ﾟдﾟ)さっきのログが出てこねえ...

## 結局

`docker-compose up ~` で起動したコンテナは `Ctrl+C` で止めただけだとそのまま残っちゃって、
`docker-compose up ~` の度にそのコンテナが起動してるからだったっぽい

## 何故今まで気づかなかったか

今までは `docker-compose up -d ~` でコンテナ起動して `docker-compose down` で止めてたんだけど、
`down` コマンドのリファレンスにはこう書かれてた。

> コンテナを停止し、 `up` で作成したコンテナ・ネットワーク・ボリューム・イメージを削除します。デフォルトではコンテナとネットワークのみ削除します。

つまり今までは意識せずにコンテナを消してたんですねー( ﾟдﾟ)

確かに、 `docker-compose up mysql` で動いてるコンテナを `Ctrl+C` で止めると

```
^CGracefully stopping... (press Ctrl+C again to force)
Stopping vagrant_mysql_1 ... done
```

と出力されるのに対し `docker-compose down` すると

```
$ docker-compose down
Stopping vagrant_mysql_1 ... done
Removing vagrant_mysql_1 ... done
Removing network vagrant_default
```

と出力されてコンテナが消されてるのが分かる。

# 結論

- `docker-compose up ~` で起動したコンテナは `Ctrl+C` で止めただけだと残り続ける
- 初期化する場合は `docker-compose down` か `docker-compose rm ~` でコンテナを消す必要がある


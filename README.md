# このリポジトリについて

このリポジトリは私が、
[HCのDockerの課題のリポジトリ](https://github.com/ihatov08/rails7_docker_template)からコピーして、
Docker上でRailsのプロジェクトを動かす練習に使ったものです。
以下はそのやり方について解説します。

## 自分のリポジトリにコピーをする
![Use This Template](/ss01.png)
Use This Templateと書かれた緑のボタンを押して、そこから自分のリポジトリとして新しく作ります。
新しく作った自分のリポジトリからローカルにクローンします。
ローカルでdockerというブランチを切ります。
以降、dockerブランチで作業します。

## ruby用のDockerfileを作る
``` 
FROM ruby:3.2.2

RUN apt-get update && apt-get install -y \
build-essential \
libpq-dev \
nodejs \
postgresql-client \
yarn

WORKDIR /docker-ruby
COPY Gemfile Gemfile.lock /docker-ruby/
RUN bundle install
COPY . /docker-ruby


```
FROMでRubyの3.2.2のイメージを取ってきます。  

RUNで apt-get updateでapt-getのリストを更新して、  
apt-get installでRailsとpostgresqlに必要なファイルをインストールしています。  

WORKDIRで作業フォルダを指定します

COPYでGemfileとGemfile.lockを作業フォルダに移してしてから、
RUN bundle installを使ってRailsをインストールします。
そのあとホストのカレントディレクトリのものをすべて、コンテナの作業フォルダにコピーします。
この順番でやることによって、Dockerのレイヤーをうまく使って、パフォーマンスがいいです。

## docker-compose.ymlを編集する

```
version: '3'

volumes:
  docker-ruby-db:

services:
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    ports:
      - "3000:3000"
    volumes:
      - ".:/docker-ruby"
    tty: true
    stdin_open: true
    environment:
      - "DATABASE_PASSWORD=postgres"
    depends_on:
      - db

  db:
    image: postgres:12
    volumes:
      - "docker-ruby-db:/var/lib/postgresql/data"
    environment:
      - "POSTGRES_PASSWORD=postgres"
```
versionで3に指定します。
```
volumes:
  docker-ruby-db:
```
これで、docker-ruby-dbというボリュームを確保します。

services:の中にwebとdbを作ります。
web側にrubyやrailsを入れて、
db側にpostgresqlを入れます。

### web:の中に書いてあるもの
```
build .
```
でカレントディレクトリでビルドをします

```
command: bundle exec rails s -p 3000 -b '0.0.0.0'
```
これは、コンテナが起動したら、実行するコマンドで
bundleからrails sでサーバーを起動して
ポートを3000番に指定して、
b- に'0.0.0.0'を指定することで、すべてからアクセス可能にします。
これを指定しないとホストからアクセスできません。

```
ports:
　- "3000:3000"
```
これを指定することで、ホストの3000番に来るアクセスをコンテナの3000番に渡します。

```
volumes:
  - ".:/docker-ruby"
```
これで手元のフォルダとコンテナ内の作業フォルダを同期させます。


```
tty: true
stdin_open: true
```
これで、デフォルトで、ターミナルからインタラクティブにコマンド打てるようにします。
いわゆる-itオプション。

```
depends_on:
  - db
```
Railsで使うDBを指定しています。
servicesのwebの下で定義したものを指定しています。

```
environment:
  - "DATABASE_PASSWORD=postgres"
```
これは環境変数を定義していますが、本番環境では別のやり方でやったほうがいいです。
この環境変数はコンテナの中に渡されます。

### db:の中に書いてあるもの
```
  image: postgres:12
```
自分でDockerfileでビルドせずに、イメージを取ってきています。
postgresの12を取ってきています。

```
  volumes:
    - "docker-ruby-db:/var/lib/postgresql/data"
```
上の方で確保したボリュームに、コンテナ内のDBの保管場所を合わせることで、
DBの中身を永続化できます。

```
  environment:
  - "POSTGRES_PASSWORD=postgres"
```
DBの方のコンテナに渡す環境変数です。
これはrootのパスワードですが、本番環境では別の渡し方をしたほうがいいです。

## railsのconfig/database.ymlを編集する
![ファイルを編集する](/ss02.png)
```
  host: db
  user: postgres
  port: 5432
  password: <%= ENV.fetch("DATABASE_PASSWORD") %>
```
デフォルトにスクリーンショットのように4行を追記します。
接続するDBとユーザー名とポート番号とパスワードを入れています
パスワードはdocker-compose.ymlで入れたものをってきます。


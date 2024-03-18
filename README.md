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

volumes:
  docker-ruby-db:
これで、docker-ruby-dbというボリュームを確保します。

services:の中にwebとdbを作ります。
web側にrubyやrailsを入れて、
db側にpostgresqlを入れます。

### web:の中に書いてあるもの


### db:の中に書いてあるもの



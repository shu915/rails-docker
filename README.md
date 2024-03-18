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

```
# Docker hands-on

## 目次
1. Docker 概要
2. Dockerインストール
3. Docker Hub
4. Docker Image
5. Docker Container
6. Dockerfile
7. docker-compose
8. docker-composeを使ったRuby on Railsアプリケーション開発入門
9. 最後にひとこと
10. 参考文献

## Docker 概要
Dockerは、コンテナ仮想化を用いてアプリケーションを開発・配置・実行するためのオープンソースソフトウェアあるいはオープンプラットフォームである。(by [wikipedia](https://ja.wikipedia.org/wiki/Docker))

#### Virtual Machine
![](https://i.imgur.com/dvc3ttZ.png)
ホストOS上にHypervisorと呼ばれる仮想化ソフト(ex. Virtual Box, Vmware)を使用し、仮想マシンと呼ばれる物理コンピュータと同じ機能を持った実行環境をソフトウェアで作成する。仮想マシンの中ではゲストOSがインストールされており、ライブラリの実行環境も揃っているのでプログラム・アプリケーション・動かすことができる。
- メリット：分離レベルが高い
- デメリット：重い

#### Container
![](https://i.imgur.com/vYZVCCW.png)

コンテナはアプリケーションの実行に必要なライブラリだけを含み、主にホストOSのカーネルを使用する。
- メリット：軽い
- デメリット：VMに比べ分離レベルが低い

## Docker使う意義
以前うまく動いていた手順が今回は何故か動かない → 実行環境の冪等性確保
自分の環境ではうまくいくが他の人の環境では動かない → ポータビリティ性の向上
インフラを作り直すたびに手順が変わってわからなくなる → 後世のコード化

## Docker Desktopインストール

公式にアクセス
https://docker.com
#### Mac
https://qiita.com/ama_keshi/items/b4c47a4aca5d48f2661c

![](https://i.imgur.com/K23cUjV.png)

1. ヘッダーメニューの「Products > Docker Desktop」をクリック
2. 「Mac with Intel chip」をクリックしDocker.dmgをダウンロード
3. Docker.dmgをクリックし展開

#### Windows
https://qiita.com/zaki-lknr/items/db99909ba1eb27803456

1. 基本的にMacと同じ手順で公式サイトからダウンロード。
2. 「Docker Desktop Installer.exe」を実行
3. ファイルをクリックし、ファイルを展開。
4. 基本的に、何もいじらずに　右下の「OK」

## Docker Hub
![](https://i.imgur.com/HuR9wyK.png)

https://hub.docker.com
Docker公式が運用しているDocker imageのレジストリで誰でもimageの配布と取得が可能。


## Docker Image

![](https://i.imgur.com/95GBvZv.png)

デフォルトでLinux OSのような環境が入っている。
実行環境を定義したファイル(テンプレート)。

#### イメージの取得
```
docker image pull <NAME[:TAG]>
docker pull <NAME[:TAG]>
```

#### イメージの一覧
```
docker image ls
docker images
```

#### イメージの削除
```
docker image rm <IMAGE ID>
```

### Let's try
1. Docker Hubに登録
2. Docker Hubでイメージの検索 (例): MySQL
3. Imageをpull
```
docker pull mysql
```
4. 一覧の確認
```
docker images
```
5. イメージの削除
```
docker image rm mysql
```

## Docker Container



- OSが入ったContainer ... ubuntu, centos, debianなど
- Web ServerやDB用のコンテナ ... Apache, Mysql, Postgresなど
- プログラミング言語の実行環境 ... python, java, rubyなど

### コマンド一覧

#### コンテナ作成・実行
```
docker container run <Image Name>
docker run <Image Name>
```
run = pull + create + start

#### コンテナ確認
現在実行中のコンテナ確認
```
docker ps
```
存在する全てのコンテナ確認
```
docker ps -a
```

#### コンテナ起動、停止
```
docker start <Container Name>
docker stop <Container Name>
```
### コンテナの作成
```
docker create --name <Your Container Name> <Image Name>
```

以下のオプションを付けることでコンテナを起動状態で維持できる
```
docker create —-name <Your Container Name> -it <Image Name> /bin/bash
```

#### コンテナ内に入る
```
docker run -it <Container Name> /bin/bash
```

### 実践
`ruby`のイメージを使ってコンテナ操作を学んでいく。まずはをpullする。
```
docker pull ruby
```

```
docker images
```

ubuntuコンテナ上でいくつかのコマンドを実行してみる。
```
docker run ruby ruby -v
```

ubuntuコンテナ内に入って操作を行う。
```
docker run -it ruby /bin/bash
```


`ruby -v`や`irb`を入力して遊んでみる。

## Dockerfile

### 命令
| 命令 | 説明 | 例 |
| -------- | -------- | -------- |
| FROM | Dockerfileのベースイメージを指定する。Dockerfileの中で最初に記述する | FROM ruby:3.0.0  |
| RUN  |  コマンドをコンテナ内で実行する。| RUN mkdir rails-app |
| WORKDIR | これ以降の`RUN`,`CMD`,`ENTORYPOINT`,`ADD`,`COPY`命令で使われるディレクトリを指定する。 | WORKDIR rails-app |
| COPY | ホストからコンテナへファイルコピー | |
| ENV | イメージ内の環境変数を設定 | ENV PATH="/usr/local" |
| ENTORYPOINT |  | ENTORYPOINT ["entrypoint.sh"] |

その他: `EXPOSE`, `MAINTAINER`, `ONBUILD`, `ADD`, `USER`


### イメージの構築

https://github.com/oshikawatkm/docker-rails-mysql/blob/main/Dockerfile

Dockerfileを編集し、以下を書く。
```
FROM ruby:3.0.0
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```


```
docker build -t railsapp .
```

```
docker run -it -p 3000:3000 railsapp 
```

## Docker Network

コンテナ上に建てたアプリケーション、ソフトウェア等を用いる際に、ネットワークの設定を行なっていなければ外部から利用することができない。
* Host ⇄ Aplication(ex.Web) Container
* Application Container ⇄ DB Container

**-pオプション**を付けて、コンテナのポートの開放を設定できる。
```
-p <HostPort>:<ContainerPort>
```
コマンドの実行例、
```
docker run --name some_container -d 8080:80
```

## Docker Volume
![](https://i.imgur.com/MQYFI4T.png)

PCの記憶領域の一部をContainerの記憶領域として割り当てる(Mountする)ことでデータの永続化をすることができる。
Volumeの割り当てをしていないとContainerの削除時にContainer内のデータも消えてしまう。

## docker-compose
![](https://i.imgur.com/wJcp9Gy.png)

コンテナの設定を`docker-compose.yml`に書くことで複数コンテナを一斉に起動、削除することが可能。

Docker Composeのステップ:
- Dockerfileを作成する or Image使う
- docker-compose.ymlを定義する
- docker-compose upを実行する


## インストール
### Mac

```
sudo curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

### Windows


`docker-compose -v`でインストールできているか確認

### Composeコマンド一覧

| コマンド | 説明 | 例 |
| -------- | -------- | -------- |
| up | Composeファイルで定義された全てのコンテナを起動する。 | docker-compose up -d |
| build | イメージを再構築する。(イメージを更新する必要がある場合) | docker-compose build |
| ps | Composeで管理しているコンテナの状態を確認する。 | docker-compose ps |
| run | コンテナを起動する。 | docker-compose run |
| stop | コンテナを停止する。 | docker-compose stop |
| rm | 停止しているコンテナを削除する。 | docker-compose rm |
| logs | ログを表示。 | docker-compose logs |

### docker-compose.yaml

Dockerの実行時に入力するコマンドをファイルに書いて、docker-composeで一度に起動・操作するイメージ。

```
version: '3'
services:
　db:
　　image: mysql:8.0
　　environment:
　　　MYSQL_ROOT_PASSWORD: password
　　ports:
　　　- '3306:3306‘
　　command: --default-authentication-plugin=mysql_native_password
　　volumes:
　　　- mysql-data:/var/lib/mysql
　web:
　　build: .
　　command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0‘“
　　volumes:
　　　- .:/myapp
　　ports:
　　　- "3000:3000"
　　depends_on:
　　　- db
　　stdin_open: true
　　tty: true
　volumes:
　　mysql-data:
　　　driver: local
```

## 実践
rails6.0 + MySQLのアプリケーソンをdocker-composeで作ってみる

1. download or git clone:
https://github.com/oshikawatkm/docker-rails-mysql

2. アプリケーションをビルドする
```
 docker-compose build
```

3. アプリケーションを起動する
```
 docker-compose up
```
4. webブラウザでlocalhostを開く
http://localhost:3000
5. dockerコンテナでコマンドを実行してみる
```
docker-compose run web rails generate scaffold user name:string age:integer
docker-compose run web rails db:migrate
```
6. 作成したuserページを開いてみる
http://localhost:3000/users
7. コンテナを終了し、削除する
```
 docker-compose down --volume
```

## 最後に一言
「Docker使うのをためらうな！」
* 環境構築が苦手な → Docker imageで構築できればいい
* 大きなデータを保存するようなソフトウェアの管理は難しい → Docker一括管理指定しまえばいい
* Dockerを使わない開発現場は無い

## 参考文献

「Docker」Adrian Mouat　著、Sky株式会社 玉川 竜司　訳
https://www.oreilly.co.jp/books/9784873117768/

「仕組みと使い方がわかる Docker&Kubernetesのきほんのきほん」著作者名：小笠原種高
https://book.mynavi.jp/ec/products/detail/id=120304

[Qiita]【図解】Dockerの全体像を理解する -前編- (by etaroid)
https://qiita.com/etaroid/items/b1024c7d200a75b992fc

[Qiita] 【Rails】Rails 6.0 x Docker x MySQLで環境構築
https://qiita.com/nsy_13/items/9fbc929f173984c30b5d


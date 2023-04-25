# EC2アプリケーションデプロイ

## EC2環境構築（事前作業）
### パッケージ更新、gitのインストール
```
$ sudo yum update
$ sudo yum install git
$ sudo yum install ImageMagick
```
### MariaDB用パッケージを削除
```
$ sudo yum list installed | grep mariadb
$ sudo yum remove mariadb-*
```

### Mysqlインストール
```
$ sudo yum localinstall https://dev.mysql.com/get/mysql80-community-release-el7-5.noarch.rpm
$ sudo yum install --enablerepo=mysql80-community mysql-community-server
$ sudo yum list installed | grep mysql
```

## rbenvの構築
### rbenvのインストール
```
$ cd /opt
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
```
### パスの設定
```
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```

### ruby-buildのインストール
```
$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
$ cd ~/.rbenv/plugins/ruby-build
$ sudo ./install.sh
```

## rubyの構築
### rubyのインストール
```
$ sudo yum -y install gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel zlib zlib-devel libffi-devel libxml2 libxslt libxml2-devel libxslt-devel sqlite-devel
$ rbenv install 3.1.2 -v
```

### rubyの有効化
```
$ rbenv global 3.1.2
$ ruby -v
```

## 各ライブラリのインストール
### bundlerのインストール
```
$ gem install bundle
```
### ライブラリインストール
サンプルアプリのディレクトリへ移動
```
$ bundle install
```
### Node.js（nvm） インストール
```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
$ nvm ls-remote
$ nvm install v16.14.0
$ nvm use 16.14.0
```
### yarnコマンドのインストール
```
$ npm install --global yarn
$ yarn --version
```

## データベースの設定
### database.yml 書き換え
```
adapter:   データベース種類
encoding:  文字コード
reconnect: 再接続するかどうか
database:  データベース名
pool:      コネクションプーリングで使用するコネクションの上限
username:  ユーザー名
password:  パスワード
host:      ホスト名(RDSのエンドポイント)
```
### データベース作成
```
$ rails db:create
$ rails db:migrate
```

## Rails関連コマンド
### Railsサーバー起動
```
$ rails s -b 0.0.0.0
```
### プリコンパイル処理
```
$ rails tmp:cache:clear
$ rails assets:precompile
```

# 構築時のエラー
## エラー①
db:createで失敗
```
$ mkdir /var/lib/mysql
$ sudo touch /var/lib/mysql/mysql.sock
$ sudo chown mysql:mysql /var/lib/mysql
```
## エラー②
```
ActionView::Template::Error (The asset "application.css" is not present in the asset pipeline.)
```
/config/environments/production.rb配下
修正箇所①
```
# config.assets.compile = false
config.assets.compile = true
```

修正箇所②
```
# config.public_file_server.enabled = true
config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?
```
修正後、プリコンパイル処理を実施
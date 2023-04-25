# Webサーバ、APサーバ構築
## Unicornインストール
/config/unicorn.rb を編集
※アクセス権限には注意
```
listen '/var/lib/unicorn/unicorn.sock' 
pid    '/var/lib/unicorn/unicorn.pid'
listen 80
```

編集後に以下のコマンドを実行
```
$ bundle exec unicorn_rails -c config/unicorn.rb -E development -D
```

起動確認
```
ps aux | grep unicorn
```

## Nginxのセットアップ
インストール
```
$ sudo yum -y install nginx
$ nginx -v
```
自動起動設定
```
$ sudo systemctl enable nginx
```
conf.d配下にファイル作成
※ファイル名は「アプリケーション名.conf」にすること
```
upstream unicorn {
  server unix:/var/lib/unicorn/unicorn.sock
  fail_timeout=0;
}
  
server {
  listen 80;
  client_max_body_size 4G;
  #server_name 54.178.10.72; ドメイン未登録

  keepalive_timeout 5;
  
  root /app/raisetech-live8-sample-app/public;
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;
  
  error_page 500 502 503 504 /500.html;
  
  try_files $uri/index.html $uri @unicorn;
  
  location @unicorn {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_pass http://unicorn;
  }
  
  #location ~ ^/assets/ {
  #  root /app/raisetech-live8-sample-app/public;
  #}
  
  #error_page 500 502 503 504 /500.html;
  #location = /500.html {
  #  root /app/raisetech-live8-sample-app/public;
  #}
}
```
起動
```
$ sudo systemctl start nginx.service 
$ ps aux | grep nginx
```

再起動
```
$ sudo systemctl restart nginx
```

# 構築時のエラー
## エラー①
Unicorn起動エラー
```
$ bundle exec unicorn_rails -c config/unicorn.rb
```
上記コマンドで起動エラーが発生、
以下のコマンドオプションを付け加えて確認再度実施し、プロセス立ち上がっていることが確認できたので、プロセスをkillし再度実施すると正常起動。
```
$ bundle exec unicorn_rails -c config/unicorn.rb -E development -D
```

## エラー②
Nginx起動エラー
```
Apr 23 15:34:00 ip-192-168-0-9.ap-northeast-1.compute.internal nginx[4664]: nginx: [emerg] "http" directive is not allowed here in /etc/nginx/default.d/RaisetechLi
Apr 23 15:34:00 ip-192-168-0-9.ap-northeast-1.compute.internal nginx[4664]: nginx: configuration file /etc/nginx/nginx.conf test failed
Apr 23 15:34:00 ip-192-168-0-9.ap-northeast-1.compute.internal systemd[1]: nginx.service: control process exited, code=exited status=1
Apr 23 15:34:00 ip-192-168-0-9.ap-northeast-1.compute.internal systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
```
修正箇所
作成した「アプリケーション名.conf」の配置場所が間違い。
conf.d配下に配置

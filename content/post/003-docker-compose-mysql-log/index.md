---
title: docker-composeで構築したMySQLのログをローカルに保存する
date: 2024-06-02
lastmod: 2024-06-14
slug: docker-compose-mysql-log
categories:
    - phpmyadmin
    - Docker
    - MySQL
---

## ファイル・フォルダ構成
以下のような構成になっています
```
dev
┣━ db-data
┣━ log
┣━ mysql
┃　 ┗━ my.cnf
┗━ docker-compose.yaml
```

- docker-compose.yaml のファイル
```
services:
  mysql:
    image: mysql:8.4.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_DATABASE: db
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      TZ: 'Asia/Tokyo'
    volumes:
      - ./db-data:/var/lib/mysql
      - ./mysql:/etc/mysql/conf.d
      - ./log:/var/log/mysql

  phpmyadmin:
    image: phpmyadmin:5.2.1
    depends_on:
      - mysql
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOSTS=mysql
      - PMA_USER=root
      - PMA_PASSWORD=mysql
    ports:
      - "3001:80"
volumes:
  db-data:
```

## docker composeで構築したMySQLのログをローカルに保存したい
my.cnfファイルに以下の内容を追記する
```
[mysqld]
log_output=FILE

# General Log
general_log=1
general_log_file=/var/log/mysql/mysql-query.log

# Slow Query Log
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log
# slow_query_time = 1.0s
long_query_time=1.0
log_queries_not_using_indexes=0

# Error Log
log_error=/var/log/mysql/mysql-error.log
log_error_verbosity=3
```
docker-compose.yamlに以下の内容を追記する
```
    volumes:
      - ./log:/var/log/mysql
```

## ログが書き込まれない
パーミッションがありすぎるとログファイルが生成されないので権限を必要最小限にする
```
sudo chmod -R 775 .
```
```
mao@mao:~/dev$ sudo ls -l ./log
total 32
-rw-r----- 1 999 systemd-journal 16239  6月  1 23:33 mysql-error.log
-rw-r----- 1 999 systemd-journal 10468  6月  1 23:29 mysql-query.log
-rw-r----- 1 999 systemd-journal   180  6月  1 23:29 mysql-slow.log
```

## docker-composeのスタートとストップ
スタート
```
sudo docker compose up -d
```
ストップ
```
sudo docker compose down -v
```
以下のURLからphpmyadminにアクセスし、少し作業をします  
するとログファイルを作成されます
- http://localhost:3001/

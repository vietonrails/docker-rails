Sử dụng docker để setup môi trường cho Rails
===

Yêu cầu đầu tiên là bạn đã có kiến thức cơ bản về `docker`. Ở đây mình sẽ chỉ hướng dẫn cách sử dụng và không giải thích nhiều. Bạn có thể tham khảo các link sau để nhận được giải thích cụ thể hơn. 
* [Cơ bản về docker](http://vietonrails.com/server/2016/06/26/co-ban-ve-docker)
* [Dùng docker để tạo môi trường rails5](http://vietonrails.com/server/2016/06/26/dung-docker-de-tao-moi-truong-dev-cho-rails5)

# Cách sử dụng

## Clone
Clone repo này: 

```
git clone git@github.com:vietonrails/docker-rails.git
```

## Chỉnh sửa `Dockerfile`
Lựa chọn version Ruby mà bạn muốn bằng cách sửa dòng số 2.

```
# Use Ruby 2.3.0, you can config your Ruby version, sample : 2.2.0, 2.2.2, ...
FROM ruby:2.3.0
```

## Chỉnh sửa `docker-compose.yml`
Lựa chọn db mà bạn muốn. Ở đây mình có 2 sample cho `mysql` và `postgres`.

```
  db:
    image: mysql
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
  # Sample for postgres
  # db:
  #   image: postgres
```

## Rails new
Quá trình chuẩn bị đã xong. Tiếp theo tạo 1 webapp mới. Chọn db engine mà bạn muốn dùng. 

```
docker-compose run web rails new . --force --database=mysql --skip-bundle
```

## docker-compose build
Build các containers liên quan bằng command `build`

```
docker-compose build
```

Kết quả nhận được như sau : 

```
db uses an image, skipping
Building web
Step 1 : FROM ruby:2.3.0
 ---> 268e5f4c6264
Step 2 : RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
 ---> Using cache
 ---> b5f055055282
Step 3 : RUN mkdir /myapp
 ---> Using cache
 ---> ef58bac5859f
Step 4 : WORKDIR /myapp
 ---> Using cache
 ---> d712516a5383
Step 5 : ADD Gemfile /myapp/Gemfile
 ---> Using cache
 ---> 45897d961453
Step 6 : ADD Gemfile.lock /myapp/Gemfile.lock
 ---> Using cache
 ---> 1e9aadcd1985
Step 7 : RUN bundle install
 ---> Running in db6cade21717
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies...
Installing rake 11.1.2
...snip...
Removing intermediate container b18b021bc248
Step 8 : ADD . /myapp
 ---> e259ae327e06
Removing intermediate container 568e85cfeee6
Successfully built e259ae327e06
```

## Config `database.yml`
Default ban đầu của Rails là `localhost` nên ta cần chỉnh lại phần `hosts` thành `db`. `db` ở đây là tên container của database. 

```
# config/database.yml
default: &default
  adapter: mysql2
  encoding: utf8
  pool: 5
  username: root
  password:
  host: db

development:
  <<: *default
  database: myapp_development
```

## Create database 

```
docker-compose run web rake db:create
```

## docker-compose up

Dùng `up` để gọi server dậy :D.

```
$ docker-compose up
Recreating rails1_db_1
Recreating rails1_web_1
Attaching to rails1_db_1, rails1_web_1
db_1  | 2016-04-09T16:48:34.775827Z 0 [Note] mysqld (mysqld 5.7.11) starting as process 1 ...
...snip...
db_1  | Version: '5.7.11'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
web_1 | [2016-04-09 16:48:38] INFO  WEBrick 1.3.1
web_1 | [2016-04-09 16:48:38] INFO  ruby 2.3.0 (2015-12-25) [x86_64-linux]
web_1 | [2016-04-09 16:48:38] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
```

## Kiểm tra IP

```
$ docker-machine ip default
192.168.99.100
```

Như vậy ta có thể vào `192.168.99.100:3000` để kiểm tra lại.

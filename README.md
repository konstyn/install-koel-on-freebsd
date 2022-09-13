### Установка koel на FreeBSD


> install nginx
> install mariadb-server

создаём базу: koel
пользователя: koel
пароль: koelkoel

> install php
 
устанавливаем  модули php

> php -m
```
[PHP Modules]
Core
ctype
curl
date
dom
exif
fileinfo
filter
gd
hash
iconv
intl
json
libxml
mbstring
mysqli
mysqlnd
openssl
pcre
PDO
pdo_mysql
pdo_sqlite
Phar
posix
Reflection
session
SimpleXML
SPL
sqlite3
standard
tokenizer
xml
xmlreader
xmlwriter
Zend OPcache
zip
zlib
```

> cat /usr/local/etc/php.ini
```
memory_limit = 512M
date.timezone = Europe/Samara
```

> cat /usr/local/etc/php-fpm.d/www.conf
```
listen = /var/run/php-fpm.sock
listen.owner = www
listen.group = www
listen.mode = 0660
```

> cat /usr/local/nginx/nginx.conf
```
user  www;
worker_processes  2;
error_log  /var/log/nginx/error.log;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include       /usr/local/etc/nginx/sites/*;
}

```

> cat /usr/local/etc/nginx/sites/koel.conf
```
server {
        listen       80;
        server_name  koel;
        root         /usr/local/www/koel/public;
        index index.html index.htm index.php;

        location / {
            try_files $uri $uri/ /index.php?q=$uri&$args;
        }

        location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {                       
           access_log off;                                                                                                                                                         
           log_not_found off;                                                                                                                                                      
           expires max; # кеширование статики                                                                                                                                      
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php-fpm.sock;
                fastcgi_index index.php;
                #fastcgi_param SCRIPT_FILENAME $request_filename;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include fastcgi_params;
                fastcgi_ignore_client_abort on;
                fastcgi_param  SERVER_NAME $http_host;
        }
}
```

> git clone https://github.com/koel/koel.git  --recursive
cd koel
npm install
nvim package.json

удаляем всё связанное с cypress

> composer install
cat .env
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=koel
DB_USERNAME=koel
DB_PASSWORD=koelkoel
```


> mkdir ~/media
php artisan koel:init --no-interaction
php artisan serve --host 0.0.0.0


переносим настроенное приложение в каталог веб сервера
перезапускаем службы

> mv koel /usr/local/www/
chown -R www:www /usr/local/www/koel
service php-fpm restart
service nginx restart

cat /etc/crontab
```
0 0 * * * cd /var/www/html/streaming/koel/ && /usr/bin/php artisan koel:sync >/dev/null 2>&1
```

default:
Username: `admin@koel.dev`  
Password: `KoelIsCool`

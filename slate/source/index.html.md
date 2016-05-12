---
title: Linux Droplet Configuration

language_tabs:
  - shell

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  

search: true
---

# Introduction

In this guide, we will discuss how to install Laravel on Ubuntu 14.04. We will be using Nginx as our web server and will be working with the most recent version of Laravel at the time of this writing, version 5.2.


# Authentication

> To authenticate via ssh, use this code:

```shell
ssh user@ip

```
replace the user and ip upon the configuration.

# Install GIT

```shell
sudo apt-get install git

```

type `git --version` in the terminal to make sure it is properly installed when it's done.


#Install Webserver

##Dependencies

>update the server packages

```shell
sudo apt-get update
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:ondrej/php5-5.6
sudo apt-get update
```

In case `apt-get update` fails and asks for a certain key:
<p>
Go to <a href="http://keyserver.ubuntu.com/">Ubuntu Key-server</a> where you can search your key (for ex: 6AF0E1940624A220). 

<aside class="notice">While searching for key pre-append the key with 0x. So search for 0x6AF0E1940624A220.</aside>

Click on the link provided in the pub section. This should take you to page containing the key. It should start with something like Public Key Server -- Get "0x6AF0E1940624A220"
Copy the whole text from the page and save it in a file (say filename is key1).
Once you have copied the data to file, Save it. Then run the following command:

`sudo apt-key add key1`

You will get an "OK" response, and you are done. Repeat the procedure of other keys that might be missing.
<p>

##Webserver

```shell
sudo apt-get install nginx-full php5-fpm php5-cli php5-mcrypt php5-mysql mysql-server dnsmasq php5 php5-common php5-curl php5-dev php5-gd php5-imagick php5-memcache php5-pspell php5-sqlite php5-xmlrpc php5-xsl php-pear libssh2-php php5-cli
```

The following commands will install all the webserver packages 
<ul>
  <li>nginx</li>
  <li>php</li>
  <li>curl</li>
  <li>mysql</li>
  <li>php extensions</li>
</ul>


MySQL will prompt you a password. it is safer not to leave it empty. in this example we will set the follwing password: `GM@$e#16`

type `php -v` and `curl --version` in the terminal to see the versions installed.


##PHP Mcrypt Extenstion

```shell
sudo apt-get install php5-mcrypt
sudo php5enmod mcrypt
```

> Make sure mcrypt.ini has: extension=mcrypt.so, if not add it

```shell
sudo vi \etc\php5\mods-available\mcrypt.ini
```

Laravel requires the php mcrypt extension.


# Install Composer

```shell
sudo curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo composer update -vvv --profile
```

Laravel utilizes Composer to manage its dependencies. So, before using Laravel, make sure you have Composer installed on your machine.

<aside class="notice">if you browse through the directories, (for ex: cd /etc) make sure to comeback to the home directory by typing `cd ~` and run the commands from there</aside>

If your server have small memory, you might have an error thrown `composer lack of memory...` 
To fix it, do the following: 

```shell
cd ~
sudo dd if=/dev/zero of=/swapfile bs=1024 count=512k
mkswap /swapfile
swapon /swapfile
```

#Configure MySQL

```shell
sudo mysql_secure_installation
```

It will prompt you a series of question:
<ul>
  <li>Change the root password? [Y/n] n</li>
  <li>Remove anonymous users? [Y/n] <b>y</b></li>
  <li>Disallow root login remotely? [Y/n] <b>n</b></li>
  <li>Remove test database and access to it? [Y/n] <b>n</b></li>
  <li>Reload privilege tables now? [Y/n] <b>y</b></li>
</ul>

> Create a database

```shell
mysql -u root -p

show databases;
CREATE DATABASE test_db;
USE test_db;
```

> Create a user and assign priviliges

```shell
CREATE USER 'test_user'@'localhost' IDENTIFIED BY '@!@##123LLkkp@';
GRANT ALL ON test_db.* TO 'test_user'@'localhost';
FLUSH PRIVILEGES;
```

> To access the database created with its user

```shell
mysql -u test_user -p test_db
show tables;
```

> Configure some variables

```shell
sudo vi /etc/mysql/my.cnf
#add the fields at under #mysqld
ft_stopword_file = ""
ft_min_word_len= 3
```
What you will be doing next is create a database, create a user, grant him priviliges and assign it to the database created.


#Domain Configuration

Create your first project in your `/home/{user}/mydomain.com` directory. Weither it's by a git pull or composer laravel installation.
Create the sites-available file and its symlink in sites enabled to point it to the domain.

```shell
#start nginx server
sudo service nginx start

#create its sites-available configuration file
cd /etc/nginx/sites-available
sudo vi mydomain.com
```


```shell
#sites-available configuration file:
server {
    listen 80;
    server_name mydomain.com;
    root /home/{user}/{mydomain.com}/public;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/{mydomain.com}-error.log error;

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_param DB_PASS "@!@##123LLkkp@";
        fastcgi_param DB_USER "test_user";
        fastcgi_param DB_NAME "test_db";
        fastcgi_param DB_HOST "localhost";
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

> Create the symlink to point it to the domain

```shell
sudo ln -s /etc/nginx/sites-available/{mydomainorIP} /etc/nginx/sites-enabled/{mydomainorIP}
```

> Restart server

```shell
sudo /etc/init.d/nginx retart
```


#Laravel Configuration

Go to your laravel directory in /home/{user}/ and run the following commands, and update the .env file

```shell
composer update
sudo chmod -Rvc 777 storage
sudo chmod -Rvc 777 bootstrap/cache
php artisan key:generate
```


#PHP exection time and upload limit

To change the php execution time limit and upload max size, do the following commands:

```shell
sudo vi /etc/php5/fpm/php.ini
#change te following variables as you need
upload_max_filesize = 2000M
post_max_size = 2100M


sudo vi /etc/nginx/nginx.conf
  http {
  #...
        client_max_body_size 2100m;
  #...
  }

sudo vi /etc/php5/fpm/php.ini
max_execution_time = 999
memory_limit = 1024M


sudo service php5-fpm reload
sudo service nginx reload

```

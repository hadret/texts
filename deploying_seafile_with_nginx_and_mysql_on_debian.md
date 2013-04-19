Deploying Seafile with nginx and MySQL on Debian Wheezy
=======================================================

This installation guide was created for Debian Wheezy and was tested only on it. However, there's high possibility that with none or minor hacking, this will also work on any other Linux distribution.

If you find any bug, error or come up with some enhancement, please submit an [issue](https://github.com/hadret/Texts/issues) or (preferably) [pull request](https://github.com/hadret/Texts/pulls).

* * *

TODO
====

_(In no particular order)._

* File/directory permissions (write/provide)
* Uninstallation/removal steps (write/provide)
* SSL certificates config (port)
* Installation in non-root domain (port)
* Find a way for PostgreSQL to work (in progress). **PostgreSQL is planned for Seafile's next release.**

* * *

AUTHORS
=======

* Filip "Hadret" Chabik <hadret@gmail.com>

_(Remember to add yourself here when pull requesting)._

* * *

LICENSE
=======

[CC BY 3.0](http://creativecommons.org/licenses/by/3.0/)

* * *

Contents
========

Following steps are going to be described in order to install and configure Seafile:

1. Installing prerequisites
2. Creating system user
3. Deploying Seafile
4. Database establishing (MySQL)
5. Setting up init script
6. Passing to reverse-proxy (nginx)


1. Installing prerequisites
===========================

If you haven't already got it (depending on installation it might be missing in Debian), install `sudo` and update system:

    apt-get update && apt-get upgrade && apt-get install sudo

Install required packages:
_(Note: you still need to install sqlite3, cause setup is using it by default and MySQL is configured later on)._

    sudo apt-get install -y python2.7 python-setuptools python-simplejson python-imaging python-flup sqlite3 


2. Creating system user
=======================

    sudo adduser --disabled-login --gecos "Seafile" seafile

3. Deploying Seafile
====================

Download Seafile for server:

    cd /home/seafile
    sudo -u seafile -H wget -c http://seafile.googlecode.com/files/seafile-server_1.6.0_x86-64.tar.gz

I'm going to follow suggested by Seafile authors directory layout:

    sudo -u seafile -H mkdir -p data seafile/installed
    sudo -u seafile -H mv seafile-server_1.6.0_x86-64.tar.gz seafile/
    cd seafile
    sudo -u seafile -H tar -xzf seafile-server_1.6.0_x86-64.tar.gz
    sudo -u seafile -H mv seafile-server_1.6.0_x86-64.tar.gz installed/

Setting Seafile up:

    cd seafile-server-1.4.5/
    sudo -u seafile -H sh setup-seafile.sh

    # More details about this step are to be found on the official installation page, section Setup:
    # URL: https://github.com/haiwen/seafile/wiki/Download-and-setup-seafile-server

My settings were following:

    server port:      10001 (default)
    seafile data dir: /home/seafile/data (different)
    seafile port:     12001 (default)
    httpserver port:  8082 (default)

I'm omitting user settings, cause they don't matter at this point -- DB setup will be overwritten by MySQL (by default Seafile is using SQLite3).

4. Database establishing MySQL
==============================

Install MySQL:

    sudo apt-get install -y mysql-server mysql-client python-mysqldb

Login to MySQL:

    mysql -u root -p

Create user:

    mysql> CREATE USER 'seafile'@'localhost' IDENTIFIED BY '$password';

    # Remember to change $password to some real, secure value

Create required databases:

    mysql> CREATE DATABASE IF NOT EXISTS `ccnet-db` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
    mysql> CREATE DATABASE IF NOT EXISTS `seafile-db` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
    mysql> CREATE DATABASE IF NOT EXISTS `seahub-db` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

Grant seafile user permissions on databases:

    mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `ccnet-db`.* TO 'seafile'@'localhost';
    mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `seafile-db`.* TO 'seafile'@'localhost';
    mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `seahub-db`.* TO 'seafile'@'localhost';
    mysql> \q

Check connections:

    sudo -u seafile -H mysql -u seafile -p -D ccnet-db
    sudo -u seafile -H mysql -u seafile -p -D seafile-db
    sudo -u seafile -H mysql -u seafile -p -D seahub-db

### Configure ccnet to use MySQL:

    cd /home/seafile
    sudo -u seafile -H vim seafile/ccnet/ccnet.conf

Append following configuration:

    [Database]
    ENGINE=mysql
    HOST=localhost
    USER=seafile
    PASSWD=$password
    DB=ccnet-db
    UNIX_SOCKET=/var/run/mysqld/mysqld.sock

    # Remember to change $password to real value

### Configure Seafile to use MySQL:

    sudo -u seafile -H vim data/seafile.conf

Replace existing database section with following:

    [database]
    type=mysql
    host=localhost
    user=seafile
    password=$password
    db_name=seafile-db
    unix_socket=/var/run/mysqld/mysqld.sock

    # Remember to change $password to real value

### Configure seahub to use MySQL:

    sudo -u seafile -H vim seafile/seahub_settings.py

Append following lines:

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'USER': 'seafile',
            'PASSWORD': '$password',
            'NAME': 'seahub-db',
            'HOST': '/var/run/mysqld/mysqld.sock',
        }
    }

    # Remember to change $password to real value

### Create DB structures:

    sudo -u seafile -H seafile/seafile-server-1.4.5/seafile.sh start

### Synchronize DBs, create tables and create superuser:

This is one long step which involves couple of operations that need to be done in this particular order. So, at first there's a switch to seafile user, next changing directory to get into seahub application main directory. Following step is passing environment settings and issuing syncdb to fill in DBs with needed tables. Lastly, there's superuser creation (this is going to be the first user for your Seafile app), leaving seafile environment, going to app install directory and launching it (this can be done cause in previous step there was seafile server started).

    sudo su - seafile
    cd seafile/seafile-server-1.4.5/seahub

    export CCNET_CONF_DIR=/home/seafile/seafile/ccnet
    export SEAFILE_CONF_DIR=/home/seafile/data
    INSTALLPATH=/home/seafile/seafile/seafile-server-1.4.5
    export PYTHONPATH=${INSTALLPATH}/seafile/lib/python2.6/site-packages:${INSTALLPATH}/seafile/lib64/python2.6/site-packages:${INSTALLPATH}/seahub/thirdpart:$PYTHONPATH
    python manage.py syncdb

    python manage.py createsuperuser
    exit
    cd /home/seafile/seafile/seafile-server-1.4.5
    sudo -u seafile -H ./seahub.sh start-fastcgi

Before you continue to next step, switch off seafile and seahub:

    sudo -u seafile -H ./seahub.sh stop
    sudo -u seafile -H ./seafile.sh stop

5. Setting up init script
=========================

    sudo wget https://github.com/hadret/Scripts/raw/master/seafile/seafile -O /etc/init.d/seafile
    sudo chmod +x /etc/init.d/seafile
    sudo update-rc.d seafile defaults 21

You may now start, stop, restart and check status of seafile via `sudo service seafile {command}` or `sudo /etc/init.d/seafile {command}`:

    sudo service seafile start

More information + additional instructions can be found on its dedicated [repository](https://github.com/hadret/Scripts/tree/master/seafile).


6. Passing to reverse-proxy (nginx)
===================================

Install nginx:

    sudo apt-get install -y nginx

Configure nginx to serve as reverse-proxy:

    sudo vim /etc/nginx/sites-available/seafile

Put and edit accordingly following content:

    server {
    listen 80;
    server_name www.myseafile.com;
    location / {
        fastcgi_pass    127.0.0.1:8000;
        fastcgi_param   SCRIPT_FILENAME     $document_root$fastcgi_script_name;
        fastcgi_param   PATH_INFO           $fastcgi_script_name;

        fastcgi_param   SERVER_PROTOCOL     $server_protocol;
        fastcgi_param   QUERY_STRING        $query_string;
        fastcgi_param   REQUEST_METHOD      $request_method;
        fastcgi_param   CONTENT_TYPE        $content_type;
        fastcgi_param   CONTENT_LENGTH      $content_length;
        fastcgi_param   SERVER_ADDR         $server_addr;
        fastcgi_param   SERVER_PORT         $server_port;
        fastcgi_param   SERVER_NAME         $server_name;

        access_log      /var/log/nginx/seahub.access.log;
        error_log       /var/log/nginx/seahub.error.log;
    }       

    location /media {
        root /home/seafile/seafile/seafile-server-1.4.5/seahub;
    }
    }

Create symbolic link to enable site in nginx and reload its configuration:

    sudo ln -s /etc/nginx/sites-available/seafile /etc/nginx/sites-enabled/seafile
    sudo service nginx reload


Credits & links:
================

*  [Seafile official documentation](https://github.com/haiwen/seafile/wiki)
*  [Seafile official webiste](http://seafile.com/en/home/)
*  [nginx official webiste](http://nginx.org/)
*  [MySQL official webiste](http://www.mysql.com/)
*  [Debian official website](http://debian.org/)

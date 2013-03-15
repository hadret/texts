Upgrading Seafile
=================

This upgrade guide assumes, that you previously followed [Deploying Seafile with nginx and MySQL on Debian Wheezy](https://github.com/hadret/Texts/blob/master/deploying_seafile_with_nginx_and_mysql_on_debian.md) and now you are upgrading to the latest stable release, 1.5.1.

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

Download latest version via:

    sudo -u seafile wget -O /home/seafile/seafile/installed/seafile-server_1.5.1_x86-64.tar.gz http://seafile.googlecode.com/files/seafile-server_1.5.1_x86-64.tar.gz

Change to deployment directory where `installed` folder inside your seafile main location:

    cd /home/seafile/seafile/installed

Stop the currently running instance, as you are about to start upgrade process:

    sudo service seafile stop

Remove older version that resides in your current folder, unpack new version and afterwards move unpacked folder one level up:

    sudo -u seafile rm seafile-server-1.4.5.tar.gz
    sudo -u seafile unp seafile-server_1.5.1_x86-64.tar.gz
    sudo -u seafile mv seafile-server-1.5.1 ..

Change to upgrade directory of your new Seafile server instance and launch the upgrade script:

    cd ../seafile-server-1.5.1/upgrade
    sudo -u seafile ./upgrade_1.4_1.5.sh

Download new version of init script for Seafile, make sure it's executable and start it:

    sudo wget https://github.com/hadret/Scripts/raw/stable-1.5.1/seafile/seafile -O /etc/init.d/seafile
    sudo chmod +x /etc/init.d/seafile
    sudo service seafile start

Update nginx config file:

    sudo vi /etc/nginx/sites-available/seafile

Change version from 1.4.5 to 1.5.1 in the location /media section and save it.
Reload nginx configuration:

    sudo service nginx reload

That's it! Enjoy your new version of Seafile! (:

Upgrading Seafile
=================

This upgrade guide assumes, that you previously followed [Deploying Seafile with nginx and MySQL on Debian Wheezy](https://github.com/hadret/Texts/blob/master/deploying_seafile_with_nginx_and_mysql_on_debian.md) and now you are upgrading to the latest stable release, 1.5.0.

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
    sudo -u seafile wget -c http://seafile.googlecode.com/files/seafile-server_1.5.0_x86-64.tar.gz /home/seafile/seafile/installed/
    cd /home/seafile/seafile/installed
    sudo service seafile stop
    Remove older version that resides 
    sudo -u seafile rm seafile-server-1.4.5.tar.gz
    sudo -u seafile unp seafile-server_1.5.0_x86-64.tar.gz
    sudo -u seafile mv seafile-server-1.5.0 ..
    cd ../seafile-server-1.5.0/upgrade
    sudo -u seafile ./upgrade_x.x_x.x.sh (./upgrade_1.4_1.5.sh)
    sudo wget https://github.com/hadret/Scripts/raw/stable-1.5.0/seafile/seafile -O /etc/init.d/seafile
    sudo chmod +x /etc/init.d/seafile
    sudo service seafile start
    sudo vi /etc/nginx/sites-available/seafile
    Change version from 1.4.5 to 1.5.0
    Reload nginx via:
    sudo service nginx reload

That's it! Enjoy your new version of Seafile! (:

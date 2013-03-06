Upgrading Seafile
=================

sudo service seafile stop
sudo -u seafile wget -c http://seafile.googlecode.com/files/seafile-server_1.5.0_x86-64.tar.gz /home/seafile/seafile/installed
cd /home/seafile/seafile/installed
rm old_version (like: rm seafile-server-1.4.x.tar.gz)
sudo -u seafile unp seafile-server_1.5.0_x86-64.tar.gz
sudo -u seafile mv seafile-server-1.5.0 ..
cd ../seafile-server-1.5.0/upgrade
sudo -u seafile ./upgrade_x.x_x.x.sh (./upgrade_1.4_1.5.sh)
sudo wget https://github.com/hadret/Scripts/raw/master/seafile/seafile -O /etc/init.d/seafile
sudo chmod +x /etc/init.d/seafile
sudo service seafile start
sudo vi /etc/nginx/sites-available/seafile
Change version from 1.4.5 to 1.5.0
Reload nginx via:
sudo service nginx reload

That's it! Enjoy your new version of Seafile!

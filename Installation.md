su

# https://linuxconfig.org/how-to-install-missing-ifconfig-command-on-debian-linux
# https://www.myblog-it.fr/2017/06/25/pas-difconfig-dans-debian-9/
apt-get install net-tools

# https://www.geek17.com/fr/content/debian-9-stretch-installer-et-configurer-sudo-61
apt-get install sudo
adduser admin-dpo sudo

# http://guillaume-cortes.fr/serveur-web-apache-debian-9/
# https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-debian-9
sudo apt install apache2

# https://tecadmin.net/install-php-debian-9-stretch/
sudo apt install ca-certificates apt-transport-https 
wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -
echo "deb https://packages.sury.org/php/ stretch main" | sudo tee /etc/apt/sources.list.d/php.list
sudo apt update
sudo apt full-upgrade
sudo apt install php7.2
sudo apt install php7.2-cli php7.2-common php7.2-curl php7.2-mbstring php7.2-mysql php7.2-xml php7.2-fpm php7.2-gd php7.2-soap php7.2-ldap php7.2-memcached php7.2-json libapache2-mod-php7.2 php7.2-zip

# https://www.lusson.fr/a-propos/ressources-utiles/article/installation-de-node-js-sur-debian-9
sudo apt install curl
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
nodejs -v
npm --version

# https://linuxize.com/post/how-to-install-mysql-on-debian-9/
cd /tmp
wget http://repo.mysql.com/mysql-apt-config_0.8.10-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.10-1_all.deb
sudo apt update
sudo apt full-upgrade
sudo apt install mysql-server

mysql -u root -p

# https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-debian-9
# https://tecadmin.net/install-php-composer-debian/
cd /tmp
sudo apt-get install git unzip
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
composer 

# https://support.qualityunit.com/263301-How-to-find-the-location-of-phpini-on-our-server-
cd /var/www/html
nano info.php
	<?php
	phpinfo();
	?>

nano /etc/php/7.2/apache2/php.ini
# http://php.net/manual/fr/session.configuration.php
# http://php.net/manual/fr/info.configuration.php
# http://php.net/manual/fr/ini.core.php

# https://linoxide.com/how-tos/disable-safe-mode-linux/
	safe_mode = Off
# https://www.system-linux.eu/index.php?post/2009/01/03/Securiser-votre-phpini
# http://www.linuxask.com/questions/turn-on-register_globals-in-php
	register_globals = Off
# https://www.developpez.net/forums/d1784352/php/outils/probleme-sessions-php-j-ai-installe-php-7-2-0-a/
# session.cache_limiter = nocache
# session.auto_start = 0
# http://php.net/manual/fr/security.magicquotes.disabling.php
	magic_quotes_gpc = Off
	memory_limit = 3000M
# https://forum.alsacreations.com/topic-20-69423-1-Augmenter-la-taille-de-uploadmaxfilesize.html
	upload_max_filesize = 128M
	post_max_size = 128M
	max_execution_time = 120
	date.timezone = "Europe/Paris"

/etc/init.d/apache2 restart

php -m
sudo apt-cache search php7.2-*

# http://www.matteomattei.com/full-web-server-setup-with-debian-9-stretch/
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod deflate
sudo a2enmod expires
service apache2 restart

# https://support.plesk.com/hc/en-us/articles/213414749-How-to-set-the-value-of-max-allowed-packet-or-wait-timeout-for-MySQL-on-a-Plesk-server-
# https://stackoverflow.com/questions/8062496/how-to-change-max-allowed-packet-size
more /etc/mysql/my.cnf
nano /etc/mysql/conf.d/mysql.cnf
	max_allowed_packet=512M

systemctl restart mysql
systemctl status mysql

# https://www.liquidweb.com/kb/create-a-mysql-database-on-linux-via-command-line/
mysql -u root -p
CREATE DATABASE dpo;
exit

cd /
git clone https://github.com/Safran/RoPA.git
mysql -u root -p dpo < /RoPA/database.sql

cd /etc/apache2/sites-available
cp 000-default.conf dpo.conf

a2dissite 000-default.conf
a2ensite dpo
systemctl reload apache2

cd /RoPA
cp .env.example .env
nano .env
	APP_URL=http://safran
	DB_DATABASE=dpo
	DB_USERNAME=root
	DB_PASSWORD=xxxxxxxxxx

composer install
npm install

npm run production
php artisan RoPA:install

# https://askubuntu.com/questions/378351/permissions-and-ownership-of-var-www
cd /RoPA
sudo chown www-data:www-data -R *
/etc/init.d/apache2 restart

# https://www.cyberciti.biz/faq/howto-linux-add-user-to-group/
sudo usermod -a -G www-data admin-dpo
cd /RoPA/public
sudo chmod g+w images
cd /RoPA/public/images
sudo chmod g+w general
sudo chmod g+w homepage
cd /RoPA/resources/assets/img
sudo chmod g+w general
# (copy logo via scp)

# https://github.com/Safran/RoPA
http://safran//fr/login

# Personnalisations
mysql -u root -p dpo 
UPDATE `companies` SET `name` = "Ma compagnie" WHERE `id`=1;
UPDATE `users` SET `first_name`="Prenom-DPO", `last_name`="Nom-DPO", `username`="Prenom-DPO.Nom-DPO", `email`="mail-DPO@Macompagnie.fr"  WHERE `id`=2;

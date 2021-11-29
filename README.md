# LinuxAdv

Step 1: Preparing your Ubuntu server
To begin with, you need a cloud server to run the LAMP stack software. If you are new to UpCloud, have a look at our quick started guide for deploying your first cloud server and how to connect to it.

Once you have your cloud server up and running and connect to it using SSH, you can get started!

First of all, ensure everything is up to date on your server:

sudo apt update
sudo apt upgrade
Now open ports 22 (for SSH), 80 and 443 and enable Ubuntu Firewall (ufw):

sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
Step 2: Installing and testing Apache2
Install Apache using apt:

sudo apt install apache2
Confirm that Apache is now running with the following command:

sudo systemctl status apache2
You should the get an output showing that the apache2.service is running and enabled.

● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-11-03 10:32:26 UTC; 1min 6s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 52943 (apache2)
      Tasks: 7 (limit: 2282)
     Memory: 11.9M
     CGroup: /system.slice/apache2.service
             ├─52943 /usr/sbin/apache2 -k start
             ├─52944 /usr/sbin/apache2 -k start
             ├─52945 /usr/sbin/apache2 -k start
             ├─52946 /usr/sbin/apache2 -k start
             ├─52947 /usr/sbin/apache2 -k start
             ├─52948 /usr/sbin/apache2 -k start
             └─52953 /usr/sbin/apache2 -k start
Once installed, test by accessing your server’s IP in your browser:

http://YOURSERVERIPADDRESS/
You should see a page with an “Apache2 Ubuntu Default” header showing that Apache2 has been installed successfully. If you do not see this, please ensure that the previous commands in this section have completed without error

Step 3: Installing and testing PHP 7.4
PHP 7.4 is the latest available right now so let’s install that along with some regularly used modules:

sudo apt install php7.4 php7.4-mysql php-common php7.4-cli php7.4-json php7.4-common php7.4-opcache libapache2-mod-php7.4
Check the installation and version:

php --version
Restart Apache for the changes to take effect:

sudo systemctl restart apache2
Create a phpinfo.php test page:

echo '<?php phpinfo(); ?>' | sudo tee -a /var/www/html/phpinfo.php > /dev/null
Test everything is working by accessing the following in your browser:

http://YOURSERVERIPADDRESS/phpinfo.php
You should see a PHP Version 7.4.3 page listing all of your PHP options. If you don’t or it tries to download a file, double-check that all of the above steps have completed without error.

Once you’ve confirmed that PHP is working correctly, delete the test file.

sudo rm /var/www/html/phpinfo.php
The information displayed in the PHP info could be used to find an attack vector against your web server so best not to leave it publicly accessible.

Step 4: Installing and securing MariaDB
MariaDB is a fork of MySQL from some of the original MySQL team and is a drop-in replacement. We’ll be using this over MySQL itself in this guide!

Install the required packages:

sudo apt install mariadb-server mariadb-client
Once installed, check it’s running correctly:

sudo systemctl status mariadb
You should see an output similar to the example below.

● mariadb.service - MariaDB 10.3.25 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-11-03 10:33:12 UTC; 4s ago
       Docs: man:mysqld(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 53554 (mysqld)
     Status: "Taking your SQL requests now..."
      Tasks: 31 (limit: 2282)
     Memory: 65.9M
     CGroup: /system.slice/mariadb.service
             └─53554 /usr/sbin/mysqld
Secure your newly installed MariaDB service:

sudo mysql_secure_installation

nexcloud install

What you'll need
An instance of Ubuntu Server 20.04

A user account with sudo privileges

Note: You can install Nextcloud on distributions other than Ubuntu Server. I prefer Ubuntu because it's one of the most widely used server platforms (it's the most used Linux distribution on Azure) and it's incredibly user-friendly.

How to install the necessary dependencies
The first thing to be done is the installation of the necessary dependencies. We'll break this into two sections. Log in to your server and access a terminal window. Install the first set of dependencies with the command:

sudo apt-get install apache2 mysql-server -y
When that completes, install the second group of dependencies with the command:

sudo apt-get install php zip libapache2-mod-php php-gd php-json php-mysql php-curl php-mbstring php-intl php-imagick php-xml php-zip php-mysql php-bcmath php-gmp -y
How to secure the MySQL database
With the dependencies out of the way, the next thing to be taken care of is securing the database server. Back at the terminal window, issue the command:

sudo mysql_secure_installation
Give MySQL a new admin password and answer the remaining questions with y (for yes).

How to create the database
Now we'll create the Nextcloud database and a database user. Log in to the MySQL console with the command:

sudo mysql -u root -p
Create the new database with the command:

CREATE DATABASE nextcloud;
Create a new user with the command:

CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'PASSWORD';
Where PASSWORD is a unique and strong password.

Give the new user the necessary permissions with the command:

GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
Flush the privileges and exit the console with the commands:

FLUSH PRIVILEGES;
exit
How to download and unpack the Nextcloud file
In order to install Nextcloud, we have to first download the necessary zipped file. To do that, issue the command:

wget https://download.nextcloud.com/server/releases/nextcloud-20.0.0.zip
Unpack that file with the command:

unzip nextcloud-20.0.0.zip
Move the newly-created nextcloud file to the Apache document root with the command:

sudo mv nextcloud /var/www/html/
Next, we'll give the Nextcloud folder the necessary ownership with the command:

sudo chown -R www-data:www-data /var/www/html/nextcloud
How to configure the web server
With the Nextcloud directory in place, we now have to make Apache aware of it. For that, we have to create a .conf file with the command:

sudo nano /etc/apache2/sites-available/nextcloud.conf
In that file, paste the following:

Alias /nextcloud "/var/www/html/nextcloud/"
<Directory /var/www/html/nextcloud/>
    Options +FollowSymlinks
    AllowOverride All
      <IfModule mod_dav.c>
        Dav off
      </IfModule>     

     SetEnv HOME /var/www/html/nextcloud
    SetEnv HTTP_HOME /var/www/html/nextcloud
</Directory>
Save and close the file. 

Enable the new site with the command:

sudo a2ensite nextcloud
We'll now enable the necessary Apache modules by issuing the command:

sudo a2enmod rewrite headers env dir mime
Finally, we'll change the PHP memory limit with the command:

sudo sed -i '/^memory_limit =/s/=.*/= 512M/' /etc/php/7.4/apache2/php.ini
Restart Apache with the command:

sudo systemctl restart apache2
How to finish the installation
Your venture into the command line is complete. Now you can open a browser and point it to http://SERVER_IP/nextcloud (where SERVER_IP is the IP address of the hosting server). You'll first be greeted by the web-based installer (Figure A).

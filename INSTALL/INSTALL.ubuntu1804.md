Installation on Ubuntu 18.04
============================

# 1. Install LAMP & dependencies

## 1.1. Install system dependencies

    $ sudo apt-get install zip unzip git gettext curl gsfonts

Some might already be installed.

## 1.2. Install MariaDB

    $ sudo apt-get install mariadb-client mariadb-server

### Secure the MariaDB installation

    $ sudo mysql_secure_installation

Especially by setting a strong root password.

## 1.3. Install Apache2

    $ sudo apt-get install apache2

### Enable modules, settings, and default of SSL in Apache

    $ sudo a2dismod status
    $ sudo a2enmod ssl
    $ sudo a2enmod rewrite
    $ sudo a2enmod headers

### Apache Virtual Host

Configure a new Apache Website

    $ sudo vi /etc/apache2/sites-available/monarc.conf

Copy/paste this configuration

    <VirtualHost _default_:80>
        ServerAdmin admin@localhost.lu
        ServerName monarc.local
        DocumentRoot /var/lib/monarc/fo/public

        <Directory /var/lib/monarc/fo/public>
            DirectoryIndex index.php
            AllowOverride All
            Require all granted
        </Directory>

        <IfModule mod_headers.c>
           Header always set X-Content-Type-Options nosniff
           Header always set X-XSS-Protection "1; mode=block"
           Header always set X-Robots-Tag none
           Header always set X-Frame-Options SAMEORIGIN
        </IfModule>

        SetEnv APP_ENV "development"
    </VirtualHost>

Save the file and enable the site

    $ sudo a2ensite monarc

## 1.4. Install PHP and dependencies

    $ sudo apt-get install php apache2 libapache2-mod-php php-curl php-gd php-mysql php-pear php-apcu php-xml php-mbstring php-intl php-imagick php-zip composer

## 1.5 Apply all changes

    $ sudo systemctl restart apache2.service



# 2. Installation of MONARC

## 2.1. MONARC source code

    $ sudo mkdir -p /var/lib/monarc/fo
    $ git clone https://github.com/monarc-project/MonarcAppFO.git /var/lib/monarc/fo
    $ cd /var/lib/monarc/fo
    $ sudo mkdir -p data/cache
    $ sudo mkdir -p data/LazyServices/Proxy
    $ sudo chmod -R g+w data
    $ composer install -o


### Back-end

The back-end is using [Laminas](https://getlaminas.org).

Create two symbolic links:

    $ mkdir -p module/Monarc
    $ cd module/Monarc
    $ ln -s ./../../vendor/monarc/core Core
    $ ln -s ./../../vendor/monarc/frontoffice FrontOffice
    $ cd ../..

There are 2 parts:

* Monarc\FrontOffice is only for MONARC;
* Monarc\Core is common to MONARC and to the back office of MONARC.


### Front-end

The frontend is an AngularJS application.

    $ mkdir node_modules
    $ cd node_modules
    $ git clone https://github.com/monarc-project/ng-client.git ng_client
    $ git clone https://github.com/monarc-project/ng-anr.git ng_anr

There are 2 parts:

* one only for MONARC: ng_client;
* one common for MONARC and the back office of MONARC: ng_anr.


## 2.2. Databases

### Create a MariaDB user for MONARC

With the root MariaDB user create a new user for MONARC:

    MariaDB [(none)]> CREATE USER 'monarc'@'%' IDENTIFIED BY 'password';
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON * . * TO 'monarc'@'%';
    MariaDB [(none)]> FLUSH PRIVILEGES;

### Create 2 databases

In your MariaDB interpreter:

    MariaDB [(none)]> CREATE DATABASE monarc_cli DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
    MariaDB [(none)]> CREATE DATABASE monarc_common DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

* monarc_common contains models and data created by CASES;
* monarc_cli contains all client risk analyses. Each analysis is based on CASES
  model of monarc_common.

### Initializes the database

    $ mysql -u monarc -ppassword monarc_common < db-bootstrap/monarc_structure.sql
    $ mysql -u monarc -ppassword monarc_common < db-bootstrap/monarc_data.sql

### Database connection

Create the configuration file:

    $ sudo cp ./config/autoload/local.php.dist ./config/autoload/local.php

And configure the database connection:

    return array(
        'doctrine' => array(
            'connection' => array(
                'orm_default' => array(
                    'params' => array(
                        'host' => 'localhost',
                        'user' => 'monarc',
                        'password' => 'password',
                        'dbname' => 'monarc_common',
                    ),
                ),
                'orm_cli' => array(
                    'params' => array(
                        'host' => 'localhost',
                        'user' => 'monarc',
                        'password' => 'password',
                        'dbname' => 'monarc_cli',
                    ),
                ),
            ),
        ),
    );



# 3. Update MONARC

Install Grunt:

    $ curl -sL https://deb.nodesource.com/setup_13.x | sudo bash -
    $ sudo apt-get install nodejs
    $ npm install -g grunt-cli

then update MONARC:

    $ ./scripts/update-all.sh -c


# 4. Create initial user

    $ php ./vendor/robmorgan/phinx/bin/phinx seed:run -c ./module/Monarc/FrontOffice/migrations/phinx.php


The username is *admin@admin.localhost* and the password is *admin*.

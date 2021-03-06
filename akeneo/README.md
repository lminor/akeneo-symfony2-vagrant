Akeneo PIM Community Standard Edition
=====================================
Welcome to Akeneo PIM Standard Edition.

This repository contains the minimal application needed to start a new project based on Akeneo PIM.
Practically, it means Akeneo PIM is declared as a dependency and will reside in the vendor directory.

If you want to contribute to Akeneo PIM, please use the PIM Community Dev repository located at
https://github.com/akeneo/pim-community-dev

Important Note: this application is not production ready and is intending for evaluation and development purposes only!

[![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/akeneo/pim-community-dev/badges/quality-score.png?s=05ef3d5d2bbfae2f9a659060b21711d275f0c1ff)](https://scrutinizer-ci.com/g/akeneo/pim-community-dev/)

Requirements
------------
 - PHP 5.4.4 or above
 - PHP Modules:
    - php5-curl
    - php5-gd
    - php5-intl
    - php5-mysql
    - php5-mcrypt
    - php-apc (opcode and data cache)
 - PHP memory_limit at least at 256 MB on Apache side and 512 MB on CLI side
 - MySQL 5.1 or above
 - Apache mod rewrite enabled
 - Java JRE (for compressing the JavaScript via YUI Compressor)

Akeneo PIM is based on Symfony 2, Doctrine 2 and [OroPlatform][3].
These dependencies will be installed automatically with [Composer][2].

Installation instructions
-------------------------
## Using Composer to create the project

This is the recommended way to install Akeneo PIM.

If you don't have Composer yet, download it following the instructions on
http://getcomposer.org/ or just run the following command:

    $ curl -s https://getcomposer.org/installer | php

### Create a Akeneo PIM project with Composer

Due to some Oro Platform limitations, you **MUST** create your database before launching composer.

    $ php composer.phar create-project akeneo/pim-community-standard ./pim-project v1.0.0-BETA2

Note that using the "--prefer-dist" option can speed up the installation by
looking into your local Composer cache.

### Initialize data and assets

    $ cd pim-project
    $ ./install.sh all prod

Note: This script can be executed several times if you need to reinit your db or redeploy your assets.
By default, this script initialize the dev environment.

Create the Apache Virtual host
------------------------------

```
<VirtualHost *:80>
    ServerName akeneo-pim.local

    DocumentRoot /path/to/your/pim/installation/web/
    <Directory /path/to/your/pim/installation/web/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/akeneo-pim_error.log

    # Possible values include: debug, info, notice, warn, error, crit, alert, emerg.
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/akeneo-pim_access.log combined
</VirtualHost>
```
Do not forget to change the "/path/to/your/pim/installation/web" to the full path to
the web directory inside your Akeneo PIM installation directory.

Now, you just have to add your host to hosts file `/etc/hosts`:

```
127.0.0.1 localhost akeneo-pim.local
```

Write permission for the HTTP server
------------------------------------

You must give write permission to the Apache user on the following directories:
- app/cache
- app/logs
- app/entities
- app/import
- app/export
- app/emails
- web/bundles
- app/uploads/product

Configure crontab
-----------------

To ensure that completeness is as up to date as possible, you can configure the following crontab
line:

*/5 * * * * php app/console pim:product:completeness-calculator > /tmp/completeness.log

In case you import data without running the versionning system in real time, you can make sure
the versionning is recalculated appropriately with this crontab line (assuming you filled the 
version pending table with the adequate information):

    */5 * * * * php app/console pim:versioning:refresh > /tmp/versioning.log

Checking your System Configuration
----------------------------------

Before starting to use your application, make sure that your system is properly
configured for a Symfony application.

Execute the `check.php` script from the command line:

    php app/check.php

If you get any warnings or recommendations, fix them before moving on.

Connect to your PIM application
-------------------------------

Go to http://akeneo-pim.local/ for production mode or http://akeneo-pim.local/app_dev.php for development mode.
Note: If you want to use development mode, do not forget to launch ./install.sh all dev

You can now connect as Akeneo administrator with the following credentials:
- login: "admin"
- password "admin"


Generating a clean database
---------------------------

By default, when you install the PIM, demo data are added to the database.

If you want to get only the bare minimum data to have a clean but functionnal pim,
just switch the following config line to false in app/config/config.yml:

```
pim_demo:
    load_data: false
```

Then relaunch the install.sh script with the db option:

$ ./install.sh db prod

Known issues
------------
 - with XDebug, the default value of max_nesting_level (100) is too low and make the ACL loading failed (which causes 403 HTTP response code on every application screen, even the login screen). A working value is 500:
`xdebug.max_nesting_level=500`

 - not enough memory can cause the JS routing bundle to fail on a segmentation fault. Please check with `php -i | grep memory` that you have enough memory according to the requirements

 - some segmentation fault can be caused as well by the circular references collector. You can disable it with the following setting in your php.ini files:
`zend.enable_gc = 0`

 - when pulling from beta1, some cache need to be removed by hand (`rm -rf var/cache/*`) otherwise you can get the following error:
`PHP Fatal error: Class 'Pim\\Bundle\\CatalogBundle\\EventListener\\TimestampableLister' not found in /var/www/pim-community-dev/app/cache/prod/appProdProjectContainer.php on line 3644`

 
[1]:  http://symfony.com/doc/2.1/book/installation.html
[2]:  http://getcomposer.org/
[3]:  http://www.orocrm.com/oro-platform 

# ushahidi-v3-installation
How to install ushahidi v3 on your localhost

    $ cd
    $ cd /Desktop
    $ git clone https://github.com/ushahidi/platform.git

#  System Requirements
To install the platform on your computer/server, the target system must meet the following requirements:

PHP version 5.4.0 or greater (PHP 7.0 not supported)
The following php extensions enabled: curl, gd, imap, json, mcrypt, mysqlnd (optional)
Composer
MySQL database version 5.5 or greater (PostgreSQL support planned)
Apache 2.2+ or nginx
These instructions are written specifically for Debian Linux or a derivative of it (Ubuntu, Mint, etc). You may have to adjust some things if you are using a different flavour of Linux.

# Set up the database
To create a database, first login to MySQL as root.

    $ mysql -u root -p
Using the `-p` is only required when your MySQL configuration requires it.
You may need to use the `-h` option to specify `localhost` or `127.0.0.1` if you are unable to connect.

Next, create a new user and database for Ushahidi. The username and database can be anything; we will use `ushahidi`for both in this example:

        CREATE DATABASE ushahidi_db;
        GRANT ALL ON ushahidi_db.* to ushahidi_user@localhost IDENTIFIED BY 'ushahidi-db-password';
        quit;
        
Now create a .env config file in the base of repository. Make sure that the database, username, and password match the database you just created.

        DB_HOST=localhost
        DB_NAME=ushahidi_db
        DB_TYPE=MySQLi
        DB_USER=ushahidi_user
        DB_PASS=ushahidi-db-password

# Set up URL rewrites
Rename `httpdocs/template.htaccess` to `httpdocs/.htaccess` (for Apache) to enable rewriting of all non-existent files to `index.php`.

If you are unable to enable rewriting, then you'll need to customize the init settings. 

    $ sudo a2enmod rewrite

Copy the `application/config/init.php` file into `application/config/environments/development/` (create this directory if it doesn't exist).
Edit init.php and set `index_file` to `"index.php"` to include index.php in your URLs

# An important note on urls, docroots and base_url

        $ sudo mkdir /var/www/myserver_ushahidi
        $ sudo mkdir /var/www/myserver_ushahidi/public_html
        $ sudo mkdir /var/www/myserver_ushahidi/logs
        $ sudo tee /var/www/myserver_ushahidi/logs/access.log
        $ sudo tee /var/www/myserver_ushahidi/logs/error.log

Enable writing to the logs, cache, and upload directories
The webserver will need write access to logs, cache and upload directories. To do this run:

        $ cd /Desktop/platform/
        $ chmod 0777 application/logs application/cache application/media/uploads

# Installing dependencies

We use Composer to manage server side dependencies. Once you have installed composer, you can run the update script with:

        $ cd
        
//Composer for php -v 5

        $ sudo apt-get install curl php5-cli git
        $ curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
        $ cd Desktop/platform/
        $ sudo bin/update
        $ cd ..
        $ sudo cp -r platform/ /var/www/myserver_ushahidi/public_html/
        $ cd /var/www/myserver_ushahidi/public_html/platform/
        $ sudo chmod 0777 application/logs application/cache application/media/uploads

# Configuring Virtual Host

        $ sudo vi /etc/apache2/sites-available/myserver.conf
Paste

        <VirtualHost *:80>
                ServerAdmin youremail
                ServerName myserver.com
                ServerAlias www.myserver.com
                DocumentRoot /var/www/myserver_ushahidi/public_html/platform/httpdocs/
                Errorlog /var/www/myserver_ushahidi/logs/error.log
                Customlog /var/www/myserver_ushahidi/logs/access.log combined
                   <Directory /var/www/myserver_ushahidi/public_html/platform>
                           AllowOverride All
                   </Directory>
        </VirtualHost>

        $ sudo a2ensite myserver.conf
        $ sudo vi /etc/hosts
Paste

        127.0.0.1	myserver.com

    $ sudo service apache2 restart

# Test on your browser Url- 
        myserver.com or myserver.com/api/v3/config

# Installing the client
Getting the client code

First, you will need a copy of the source code, which lives in our Github repository:

        $ git clone https://github.com/ushahidi/platform-client.git
# Client dependencies
First you'll need nodejs or io.js installed, npm takes care of the rest of our dependencies.

-nodejs v6

 We recommend using NodeJS builds from NodeSource or using NVM
 Build tools for building native addons from npm:
 Debian users install the `build-essential` package
 Fedora users install `gcc-c++` and `make`
 nginx or apache2

# Building the client
Clone the repo

    $ git clone https://github.com/ushahidi/platform-client.git
    
Note: if you're getting set up for development, you might want to fork the repository first.
Navigate to project root

    $ cd platform-client
    
Install Build Requirements

    $ sudo npm install -g gulp
    
Install Packages

    $ sudo npm install
    
This will install both NPM and Bower dependencies! No separate `bower install` command is required.
Set up build options. Create a `.env` file, you'll need to point BACKEND_URL at an instance of the platform api

    BACKEND_URL=http://myserver.com
    
Run gulp

    $ sudo gulp build
    
# Configure nginx or apache
Apache2:

Copy server/rewrite.htaccess to server/www/.htaccess

        $ sudo mkdir /var/www/ushahidi
        $ sudo mkdir /var/www/ushahidi/public_html
        $ sudo mkdir /var/www/ushahidi/logs
        $ sudo tee /var/www/ushahidi/logs/access.log
        $ sudo tee /var/www/ushahidi/logs/error.log

    $ sudo cp -r platform-client/ /var/www/ushahidi/public_html/
    $ sudo vi /etc/apache2/sites-available/ushahidi.conf
Paste

        <VirtualHost *:80>
                ServerAdmin youremail
                ServerName ushahidi.com
                ServerAlias www.ushahidi.com
                DocumentRoot /var/www/ushahidi/public_html/platform-client/server/www
                Errorlog /var/www/ushahidi/logs/error.log
                Customlog /var/www/ushahidi/logs/access.log combined
                   <Directory /var/www/ushahidi/public_html/platform-client>
                           AllowOverride All
                   </Directory>
        </VirtualHost>

    $ sudo a2ensite ushahidi.conf
    $ sudo vi /etc/hosts
Paste

127.0.0.1	ushahidi.com

    $ sudo service apache2 restart

Test on your browser Url- ushahidi.com 

# Logging in the first time

The default install creates a user admin with password admin. Once logged in this user can create further user accounts or give others admin permissions too.

# OSX Phabricator Installation Guide

### Create Sites folder and install Dependencies 

Note, please replace "~/Sites" with where ever you store your web tools.

```
$ mkdir ~/Sites
$ cd ~/Sites
$ git clone https://github.com/phacility/phabricator.git
$ git clone https://github.com/phacility/libphutil.git
$ git clone https://github.com/phacility/arcanist.git
```
### Add Arcanist to your PATH and test it out

 1. Open your ~/.profile or ~/.bashrc or whatever you use
   1. Add the following line
   1. `export PATH="$HOME/Sites/arcanist/bin:$PATH"` 
   1. Save the file 
 1. Back in the terminal 
   1. Reload your profile/bashrc `source ~/.profile`
   1. Test it out `arc help`

### Setup Phabricator

First, update the httpd.conf file and hosts file to let Apache find your Phabricator installation.

 1. Edit httpd.conf `$ sublime private/etc/apache2/httpd.conf`
   1. Add the following rule, changing the `DocumentRoot` to match location of your Phabricator repo (remember to replace \<your-user-name-here\> with the name of the user under which profile you install, e.g. johndoe)
     ```
     ## Allow access to the entire Sites directory
     <Directory "/Users/<your-user-name-here>/Sites">
          Options Indexes MultiViews FollowSymLinks
          AllowOverride All
          Require all granted
     </Directory>
     <VirtualHost *:80>
          # Change this to the domain which points to your host.
          ServerName "phabricator.local.example.com"   
          ServerAlias "www.phabricator.local.example.com"
          # Change this to the path where you put 'phabricator' when you checked it
          # out from GitHub when following the Installation Guide.
          #
          # Make sure you include "/webroot" at the end!
          DocumentRoot /Users/<your-user-name-here>/Sites/phabricator/webroot  

          RewriteEngine on
          RewriteRule ^/rsrc/(.*)     -                       [L,QSA]
          RewriteRule ^/favicon.ico   -                       [L,QSA]
          RewriteRule ^(.*)$          /index.php?__path__=$1  [B,L,QSA]
     </VirtualHost>

     ServerName localhost
     ```
   2. NOTE: If you get a warning "Invalid command 'RewriteEngine', perhaps misspelled or defined by a module not included in the server configuration", make sure that you have the following module enabled in your httpd.conf file: LoadModule rewrite_module libexec/apache2/mod_rewrite.so 
   3. NOTE: For Apache 2.4 and in all *.conf files (e.g. httpd-vhosts.conf, http.conf, httpd-autoindex.conf ..etc) use
     ```
     Require all granted
     ```
   instead of
     ```
     Order allow,deny
     Allow from all
     ```
     The Order and Allow directives are deprecated in Apache 2.4.
     
 3. Edit your hosts file `$ sublime /private/etc/hosts` 
   1. Add the following

     ```
       127.0.0.1  phabricator.local.example.com
     ```
 4. Restart apache `$ sudo apachectl -k restart`

 

### Finish Phabricator Setup

 1. Point your browser to [phabricator.local.example.com](http://phabricator.local.example.com)
 2. If you see errors connecting to MySQL, then:
   1. Make sure you have mysql `brew install mysql` and follow these directions http://stackoverflow.com/a/6378429/527096
   2. Update Phabricator mysql.config with correct root, user, and pass
      ```
      $ cd ~/Sites/phabricator
      $ ./bin/config set mysql.host local.example.com
      $ ./bin/config set mysql.user root
      $ ./bin/config set mysql.pass ""
      ```
   3. Refresh in your browser and continue following instructions

 3. If you see
   1. Important Setup Issues
     1. Base URI Not Configured: Follow the hyperlinked text of the warning and run the command shown.
     2. Phabricator Daemons Are Not Running: Follow the hyperlinked text of the warning and run the command shown.

 4. If you see 
   1. MySQL Setup Issues: Note, if you have installed MariaDB, look for https://mariadb.com/kb/en/mariadb/configuring-mariadb-with-mycnf/	
   One way to find out where your mysql configuration file is (my.cnf) is to execute the following command:
      ```
sudo /usr/libexec/locate.updatedb
# wait a few minutes for it to finish
locate my.cnf
      ```
   It may be worth creating a file ~/.my.cnf with all requirements of Phabricator (e.g. max_allowed_packet = 32M). This file will overwrite all other MySQL / MariaDB configuration files, as it is local to the user.
      
     1. Small MySQL "max_allowed_packet"
      ```
      MySQL is configured with a small 'max_allowed_packet' (1048576), which may cause some large writes to fail. Strongly   consider raising this to at least 33554432 in your MySQL configuration.
      ```
      Correct the section [mysqld] inside /etc/my.cnf from 1M (=1048576) to 32M (=33554432)
      
      max_allowed_packet = 32M
      
     2. MySQL is Using Default Stopword File
      You can choose to ignore this warning.
      
   2. Other Setup Issues: Server Timezone Not Configured
     1. Click on the hyperlinked text of the warning, then click on 'Edit phabricator.timezone'
     2. Enter the required timezone (e.g. Europe/London) in the input field and confirm with 'Save Config Entry'
     
   3. Other Setup Issues: Large File Storage Not Configured
     1. Follow the provided link 'Configuring File Storage'
     2. For each php.ini (in /usr/local/etc/php/\<version\>), set post_max_size to 32M
     3. For each php.ini (in /usr/local/etc/php/\<version\>), set upload_max_filesize to 32M
     
   4. Other Setup Issues: PHP Extension 'APC' Not Installed
     1. more ... 

### Installing `pcntl`

Phabricator requires the [pcntl](http://php.net/manual/en/pcntl.installation.php) php extension.

```
cd ~/Downloads
# what version of PHP are you running? Replace "5.4.30" with your own php version
php -v 
# Download PHP source
wget -O php-5.4.30.tar.gz http://php.net/get/php-5.4.30.tar.gz/from/this/mirror
# Deflate source 
tar -zxvf php-5.4.30.tar.gz
# Install autoconf if not already installed
brew install autoconf
# Make + install pcntl 
cd php-5.4.30/ext/pcntl/
phpize && ./configure && make install
# Copy the install into your php extensions directory 
sudo cp modules/pcntl.so /usr/lib/php/extensions/
# Update php.ini with new extension
# For each php.ini in /usr/local/etc/php/<version>
# Add below line to the php.ini
extension=/usr/lib/php/extensions/pcntl.so
# Restart and test
sudo apachectl restart
/usr/bin/php --ri pcntl
# It should say "pcntl support => enabled"
```

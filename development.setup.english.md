# Disclaimer

This setup works fine on my macOS machines. I am certainly no Apache, PHP and MySQL expert so should something go wrong in your setup, check the sources I used.

# Requirements

- macOS 10.12 or higher
- <a href="https://developer.apple.com/xcode/" target="_blank">XCode</a> installed

# Used sources

<a href="https://getgrav.org/blog/macos-sierra-apache-multiple-php-versions" target="_blank">macOS 10.13 High Sierra Apache Setup: Multiple PHP Versions</a>

<a href="https://coolestguidesontheplanet.com/get-apache-mysql-php-phpmyadmin-working-osx-10-10-yosemite/#phpmyadmin" target="_blank">Get Apache, MySQL, PHP and phpMyAdmin working on OSX 10.10 Yosemite</a>

<a href="http://www.hatsaplenty.com/2016/07/access-mail-spool-osx-postfix-dovecot/" target="_blank">Access the Mail Spool on OSX with Postfix and Dovecot</a>

# Install XCode Command Line Tools

```
$ xcode-select --install
```

# Homebrew installation

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Check Homebrew installation

```
$ brew --version
Homebrew 1.4.1
Homebrew/homebrew-core (git revision 859b; last commit 2017-12-25)
```

### Add extra Brew Taps

```
$ brew tap homebrew/php
```

### Install wget

```
$ brew install wget
```

### Stop existing Apache and install Homebrew version

```
$ sudo apachectl stop
$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
$ brew install httpd
```

You can setup Apache so it starts at every (re)boot of your machine.

```
$ sudo brew services start httpd
```

You can also start Apache manually

```
$ sudo apachectl start
```

Test Apache by going in your browser to: http://localhost:8080

### Modify Apache configuration

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Replace:

```
Listen 8080
```

With:

```
Listen 80
```

Replace:

```
DocumentRoot "/usr/local/var/www"
```

With:

```
DocumentRoot /Users/your_user/Development/Sites (so WITHOUT quotes)
```

(input your user name at 'your_user')

Replace:

```
<Directory "/usr/local/var/www">
```

With:

```
<Directory /Users/your_user/Development/Sites> (so WITHOUT quotes)
```

(input your user name at 'your_user')

In the same <Directory block:

Replace:

```
AllowOverride None
```

With:

```
AllowOverride All
```

Search for:

```
#LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

And replace with:

```
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so (dus # weghalen)
```

### Modify User & Group

Replace:

```
User _www
Group _www
```

With:

```
User your_user
Group staff
```

(input your user name at 'your_user')

### Setup servername

Replace:

```
#ServerName www.example.com:8080
```

With:

```
ServerName localhost
```

Save the file /usr/local/etc/httpd/httpd.conf

### Create a standard index.html

```
$ mkdir -p ~/Development/Sites
$ echo "<h1>My User Web Root</h1>" > ~/Development/Sites/index.html
```

### Restart Apache

```
$ sudo apachectl -k restart
```

In your browser go to http://localhost, there the My User Web Root should appear.

# Install PHP

### Install PHP 5.6

```
$ brew install php56 --with-httpd
```

### Install PHP 7.0

```
$ brew unlink php56
$ brew install php70 --with-httpd
```

### Install PHP 7.1

```
$ brew unlink php70
$ brew install php71 --with-httpd
```

### Install PHP 7.2

```
$ brew unlink php71
$ brew install php72 --with-httpd
```

### Switch back to PHP 5.6

```
$ brew unlink php72
$ brew link php56
```

# Setup PHP in Apache

### Modify Apache configuration

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Replace:

```
LoadModule php5_module        /usr/local/Cellar/php53/5.3.29_8/libexec/apache2/libphp5.so
LoadModule php5_module        /usr/local/Cellar/php56/5.6.32_8/libexec/apache2/libphp5.so
LoadModule php7_module        /usr/local/Cellar/php70/7.0.25_17/libexec/apache2/libphp7.so
LoadModule php7_module        /usr/local/Cellar/php71/7.1.11_22/libexec/apache2/libphp7.so
LoadModule php7_module        /usr/local/Cellar/php72/7.2.0RC5_8/libexec/apache2/libphp7.so
```

With:

```
#LoadModule php5_module    /usr/local/opt/php53/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php72/libexec/apache2/libphp7.so
```

Replace:

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

With:

```
<IfModule dir_module>
    DirectoryIndex kick.php index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Save the file and restart Apache:

```
$ sudo apachectl -k stop
$ sudo apachectl start
```

Create a file info.php in your ~/Development/Sites folder with these contents:

```php
<?php phpinfo();
```

Check if it is working by going in your browser to http://localhost/info.php

# Install PHP Switcher script

To easily switch between PHP versions we install a PHP switcher script.

```
$ curl -L https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

### Modify Apache configuration

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Replace:

```
#LoadModule php5_module    /usr/local/opt/php53/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php72/libexec/apache2/libphp7.so
```

With:

```
#Brew PHP LoadModule for `sphp` switcher23.1605
LoadModule php5_module /usr/local/lib/libphp5.so
#LoadModule php7_module /usr/local/lib/libphp7.so
```

Save the file. Test the switcher script:

```
$ sphp 70
```

Refresh the page <a href="http://localhost/info.php" target="_blank">http://localhost/info.php</a> in your browser.

# Modify PHP.ini

To have webapplications work good we need to modify a number of php.ini settings.
The following values need to be modified. Search the setting in php.ini, copy the line and add a ; at the beginning of the line. Then enter the new value.

For display_errors you might want make an exception and leave that to 'On', but that is however you prefer.

```
output_buffering = Off
max_execution_time = 180
max_input_time = 180
memory_limit = 256M
display_errors = Off
post_max_size = 50M
upload_max_filesize = 50M
date.timezone = Europe/Amsterdam
```

Modify php.ini PHP 5.3:

```
$ open -e /usr/local/etc/php/5.3/php.ini
```

Modify php.ini PHP 5.6:

```
$ open -e /usr/local/etc/php/5.6/php.ini
```

Modify php.ini PHP 7.0:

```
$ open -e /usr/local/etc/php/7.0/php.ini
```

Modify php.ini PHP 7.1:

```
$ open -e /usr/local/etc/php/7.1/php.ini
```

Modify php.ini PHP 7.2:

```
$ open -e /usr/local/etc/php/7.2/php.ini
```

Restart Apache after the php.ini modifications:

```
$ sudo apachectl -k restart
```

# MySQL installation

```
$ brew update
$ brew install mysql
```

### Start MySQL

```
$ brew services start mysql
```

### Secure MySQL

```
$ /usr/local/bin/mysql_secure_installation
```

Answers:

```
Would you like to setup VALIDATE PASSWORD plugin? Press y|Y for Yes, any other key for No: Enter (= no)
Please set the password for root here. New password: root
Re-enter new password: root
Remove anonymous users? y
Disallow root login remotely? y
Remove test database and access to it? y
Reload privilege tables now? y
```

# phpMyAdmin installation

Download the English phpMyAdmin at http://www.phpmyadmin.net/home_page/downloads.php

Unzip the file and move the folder to the subfolder 'phpmyadmin' below Sites, so to: ~/Development/Sites/phpmyadmin

### Run Setup in the browser

In your browser go to http://localhost/phpmyadmin/setup/

- Click on New Server
- Tab Basic settings: change nothing
- Tab Authentication:
    - at Authentication type choose: config
    - at Password for config auth enter: root
    - click Apply, you go back to Overview
- Click download and save config.inc.php in the folder ~/Development/Sites/phpmyadmin

Go in your browser to http://localhost/phpmyadmin. You should now see phpMyAdmin with the databases in the left column.

# Apache Virtual Hosts

### Modify Apache configuration

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Replace:

```
#LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

With:

```
LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
```

Replace:

```
#Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

With:

```
Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Modify httpd-vhosts.conf:

```
$ open -e /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

Remove all existing lines below the comments block and add the following lines:

```
<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Development/Sites"
    ServerName localhost
</VirtualHost>

<Directory "/Users/your_user/Development/Sites">
    Allow From All
    AllowOverride All
    Options +Indexes
    Require all granted
</Directory>

<Virtualhost *:80>
    VirtualDocumentRoot "/Users/your_user/Development/Sites/%1"
    ServerAlias *.test
    UseCanonicalName Off
</Virtualhost>
```

(input your user name at 'your_user')

# Dnsmasq installation

We can now very easily add a new virtual host.
By creating a subfolder in ~/Development/Sites/, for example, 'testsite', this new website is immediately accessible through the domain name 'testsite.test'.

But we need to modify DNS so it resolves to this domain. Therefor we install Dnsmasq.

```
$ brew install dnsmasq
```

Setup *.test hosts:

```
$ echo 'address=/.test/127.0.0.1' > /usr/local/etc/dnsmasq.conf
```

Start Dnsmasq and make sure it starts at every reboot:

```
$ sudo brew services start dnsmasq
```

Add to resolvers:

```
$ sudo mkdir -v /etc/resolver
$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/test'
```

Test this by pinging to an unknown .test name. There should come a reply from 127.0.0.1:

```
$ ping newwebsite.test
```

Restart apache:

```
$ sudo apachectl -k restart
```

# APC Cache and XDebug installation:

To have PHP run faster we install Zend OPcache and APCu Cache.

```
$ sphp 53
$ brew install php53-opcache php53-apcu --build-from-source
$ brew install php53-xdebug --build-from-source
```

Repeat this process for the other PHP versions.

```
$ sphp 56
$ brew install php56-opcache php56-apcu --build-from-source
$ brew install php56-xdebug --build-from-source
```

```
$ sphp 70
$ brew install php70-opcache php70-apcu --build-from-source
$ brew install php70-xdebug --build-from-source
```

```
$ sphp 71
$ brew install php71-opcache php71-apcu --build-from-source
$ brew install php71-xdebug --build-from-source
```

```
$ sphp 72
$ brew install php72-opcache php72-apcu --build-from-source
$ brew install php72-xdebug --build-from-source
```

Restart apache:

```
$ sudo apachectl -k restart
```

Install XDebug enable/disable script:

```
$ brew install xdebug-osx
```

The standard configurations might not be good enough. We modify the configurations.

PHP 5.3:

```
$ open -e /usr/local/etc/php/5.3/conf.d/ext-xdebug.ini
```

Replace the contents with these contents:

```
[xdebug]
zend_extension="/usr/local/opt/php53-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

PHP 5.6:

```
$ open -e /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
```

Replace the contents with these contents:

```
[xdebug]
zend_extension="/usr/local/opt/php56-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

PHP 7.0:

```
$ open -e /usr/local/etc/php/7.0/conf.d/ext-xdebug.ini
```

Replace the contents with these contents:

```
[xdebug]
zend_extension="/usr/local/opt/php70-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

PHP 7.1:

```
$ open -e /usr/local/etc/php/7.1/conf.d/ext-xdebug.ini
```

Replace the contents with these contents:

```
[xdebug]
zend_extension="/usr/local/opt/php71-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

PHP 7.2:

```
$ open -e /usr/local/etc/php/7.2/conf.d/ext-xdebug.ini
```

Replace the contents with these contents:

```
[xdebug]
zend_extension="/usr/local/opt/php72-xdebug/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_handler=dbgp
xdebug.remote_port=9000
```

If you work with PhpStorm it is also a good idea to add the following line:

```
xdebug.file_link_format="phpstorm://open?file=%f&line=%l"
```

Restart Apache:

```
$ sudo apachectl -k restart
```

You can now easily switch XDebug on or off.
Switch XDebug on for the active PHP version:

```
$ xdebug-toggle on
```

Switch XDebug off for the active PHP version:

```
$ xdebug-toggle off
```

# Startdevelopment, Stopdevelopment and Restartdevelopment

Personally I don't want Apache and MySQL te start at every (re)boot automatically.
For starting, stopping and restarting Apache and MySQL I use these three simple scripts:

```
$ touch /usr/local/bin/startdevelopment
$ open -e /usr/local/bin/startdevelopment
```

Add the following code:

```
#!/bin/bash

# Start apache
sudo apachectl start

# Start mysql
/usr/local/bin/mysql.server start

# Check running processes
ps -aef | grep httpd
ps -aef | grep mysql
```

Save the file.

```
$ touch /usr/local/bin/stopdevelopment
$ open -e /usr/local/bin/stopdevelopment
```

Add the following code:

```
#!/bin/bash

# Start mysql
/usr/local/bin/mysql.server stop

# Start apache
sudo apachectl stop

# Check running processes
ps -aef | grep httpd
ps -aef | grep mysql
```

Save the file.

```
$ touch /usr/local/bin/restartdevelopment
$ open -e /usr/local/bin/restartdevelopment
```

Add the following code:

```
#!/bin/bash

# Restart apache
sudo apachectl -k restart

# Restart mysql
/usr/local/bin/mysql.server restart

# Check running processes
ps -aef | grep httpd
ps -aef | grep mysql
```

Save the file.

Modify file rights:

```
$ chmod +x /usr/local/bin/startdevelopment /usr/local/bin/stopdevelopment /usr/local/bin/restartdevelopment
```

### Local mailserver

Redirect all mail to one address that you can control.

```
$ sudo vi /etc/postfix/main.cf
```

Add the following to the end of the file 

```
virtual_maps=regexp:/etc/postfix/virtual-redirect
```

Create the redirect file

```
$ sudo vi /etc/postfix/virtual-redirect
```

Add the following into the file

```
/.+@.+/ username
```

Replace `username` with your OSX username. You can find this username by running

```
$ whoami
```

Map this file to Postfix

```
$ sudo postmap /etc/postfix/virtual-redirect
```

Start Postfix

```
$ sudo postfix start
```

Install Dovecot

```
$ brew install dovecot
```

Copy the configuration files

```
$ cp -pr /usr/local/Cellar/dovecot/2.2.33.2/share/doc/dovecot/example-config/ /usr/local/etc/dovecot/
```

The version number `2.2.33.2` may be different. Check the Dovecot version number

```
$ brew info dovecot
```

Create the Dovecot local configuration file

```
touch /usr/local/etc/dovecot/local.conf
```

Open the local configuration

```
$ sudo open -e /usr/local/etc/dovecot/local.conf
```
Add the following configuration

```
# Listen for localhost
listen = 127.0.0.1
 
# Use IMAP
protocols = imap

# Set the authentication
passdb {
  args = login
  driver = pam
}

# Set the mail location. %u will be substituted with your username.
# The first path is where your other IMAP folders will go,
# the second is where your mail spool is.
# See dovecot/conf.d/10-mail.conf for more information.
mail_location = mbox:/Users/%u/mail:INBOX=/var/mail/%u
 
# Set the user and group for accessing mail.
mail_uid = YOURUSERNAME
mail_gid = staff
 
# Login user is internally used by login processes. This is the most 
# untrusted user in Dovecot system. It shouldn't have access to anything 
# at all.
default_login_user = _dovenull
 
# Internal user is used by unprivileged processes. It should be separate 
# from login user, so that login processes can't disturb other processes.
default_internal_user = _dovecot
 
# Group to enable temporarily for privileged operations. Currently this is
# used only with INBOX when either its initial creation or dotlocking 
# fails. Typically this is set to "mail" to give access to /var/mail.
mail_privileged_group = mail

# Add a log
log_path = /var/log/dovecot.log
```

Replace the following settings

```
YOURUSERNAME with the OSX username

```

Turn off SSL

```
$ sudo open -e /usr/local/etc/dovecot/conf.d/10-ssl.conf
```

Replace:

```
# ssl = yes
```

With:

```
ssl = no
```

Replace:

```
ssl_cert = </etc/ssl/certs/dovecot.pem
ssl_key = </etc/ssl/private/dovecot.pem
```

With:

```
#ssl_cert = </etc/ssl/certs/dovecot.pem
#ssl_key = </etc/ssl/private/dovecot.pem
```

In case you are using or upgrading to Dovecot 2.3.1 it is necessary to add the following line as well:

```
default_internal_group = mail
```

To find out which Dovecot version you are running, run this command:

```
dovecot --version
```

Set the permission on the mail folder. This is needed because otherwise Dovecot can't delete the messages and the log
shows the error imap(YOURUSERNAME): Error: setegid(privileged) failed: Operation not permitted. This means the 
/var/mail folder cannot be accessed by you because it is owned my root. We may need to setup LMTP as described here:
https://wiki2.dovecot.org/HowTo/PostfixDovecotLMTP

```
$ sudo chown YOURUSERNAME:mail /var/mail
```


Set the permissions

```
$ chmod 0600 /var/mail/*
```

Update the permissions on your mail spool with

```
$ chmod +t /var/mail/YOURUSERNAME
```

Start or Restart Dovecot
```
$ sudo brew services start dovecot

or

$ sudo brew services restart dovecot
```

### Configure the mail client

Incoming mail

```
Server Name: localhost
Port: 143
User Name: The OSX username
Connection security: None
Authentication method: Password, transmitted insecurely
User Name: YOURUSERNAME
```

Outgoing mail

```
Server Name: localhost
Port: 25
User Name: The OSX username
Connection security: None
Authentication method: Password, transmitted insecurely
User Name: YOURUSERNAME
```

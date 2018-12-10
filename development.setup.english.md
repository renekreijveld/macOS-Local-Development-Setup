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
Homebrew 1.6.2
Homebrew/homebrew-core (git revision 15ab; last commit 2018-04-30)
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

You can also start, stop or restart Apache manually

```
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl -k restart
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
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so (so remove #)
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
$ brew install php@5.6
```

### Install PHP 7.1

```
$ brew install php@7.1
```

### Install PHP 7.2

```
$ brew install php@7.2
```

### Install PHP 7.3

```
$ brew install php@7.3
```

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

Modify php.ini PHP 5.6:

```
$ open -e /usr/local/etc/php/5.6/php.ini
```

Modify php.ini PHP 7.1:

```
$ open -e /usr/local/etc/php/7.1/php.ini
```

Modify php.ini PHP 7.2:

```
$ open -e /usr/local/etc/php/7.2/php.ini
```

Modify php.ini PHP 7.3:

```
$ open -e /usr/local/etc/php/7.3/php.ini
```

Restart Apache after the php.ini modifications:

```
$ sudo apachectl -k restart
```

Switch back yo the first PHP version

```
$ brew unlink php@7.2 && brew link --force --overwrite php@5.6
```

At this point, I strongly recommend closing ALL your terminal tabs and windows. This will mean opening a new terminal to continue with the next step. This is strongly recommended because some really strange path issues can arise with existing terminals.

Quick test that we're in the correct version:

```
$ php -v
PHP 5.6.36 (cli) (built: Apr 26 2018 22:02:57) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```

# Apache PHP Setup

### Modify Apache configuration

```
$ open -e /usr/local/etc/httpd/httpd.conf
```

Find the line that loads the mod_rewrite module:

LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so

Below this add the following libphp modules:

```
LoadModule php5_module /usr/local/opt/php@5.6/lib/httpd/modules/libphp5.so
#LoadModule php7_module /usr/local/opt/php@7.1/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.2/lib/httpd/modules/libphp7.so
#LoadModule php7_module /usr/local/opt/php@7.3/lib/httpd/modules/libphp7.so
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
$ curl -L https://gist.github.com/roland-d/949342f2db21071bcefe6666e3394b4e/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

# Check your local path

```
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

If you don't see this, you might need to add these manually to your path. Depending on your shell your using, you may need to add this line to ~/.profile, ~/.bash_profile, or ~/.zshrc. We will assume you are using the default bash shell, so add this line to a your .profile (create it if it doesn't exist) file at the root of your user directory:

```
$ open -e ~/.profile
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

Then stop and restart the Terminal application.

### Testing the PHP Switching

Test the switcher script:

```
$ sphp 7.1
```

Refresh the page <a href="http://localhost/info.php" target="_blank">http://localhost/info.php</a> in your browser.

# Updating software

Brew makes it super easy to update PHP and the other packages you install. The first step is to update Brew so that it gets a list of available updates:

```
$ brew update
```

This will spit out a list of available updates, and any deleted formulas. To upgrade the packages simply type:

```
$ brew upgrade
```

To update all of your PHP versions you have to switch to them, and then run brew update.

```
$ sphp 5.6
Switching to php@5.6
Switching your shell
Unlinking /usr/local/Cellar/php@5.6/5.6.36... 0 symlinks removed
Unlinking /usr/local/Cellar/php@7.0/7.0.30... 0 symlinks removed
Unlinking /usr/local/Cellar/php@7.1/7.1.17... 0 symlinks removed
Unlinking /usr/local/Cellar/php/7.2.5... 24 symlinks removed
Linking /usr/local/Cellar/php@5.6/5.6.36... 25 symlinks created

If you need to have this software first in your PATH instead consider running:
  echo 'export PATH="/usr/local/opt/php@5.6/bin:$PATH"' >> ~/.bash_profile
  echo 'export PATH="/usr/local/opt/php@5.6/sbin:$PATH"' >> ~/.bash_profile
You will need sudo power from now on
Switching your apache conf
Restarting apache

PHP 5.6.36 (cli) (built: Apr 26 2018 22:02:57) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies

All done!
$ brew update
Already up-to-date.
```

Repeat these steps for PHP 7.1, 7.2 and 7.3.

# MySQL installation

Do note, you cannot run multiple versions of MySQL on the same machine because brew will install the database directory 
in the same location. So choose wisely which version you want to install.

### Install MySQL 8.0

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
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
```

### Install MySQL 5.7

All files for MySQL 5.7 will be located in the folder `/usr/local/opt/mysql@5.7/bin`

```
$ brew update
$ brew install mysql@5.7
```

### Start MySQL

```
$ brew services start mysql@5.7
```

### Secure MySQL

```
$ /usr/local/opt/mysql@5.7/bin/mysql_secure_installation
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

# APC Cache installation:

To have PHP run faster we install Zend OPcache and APCu Cache.

```
$ pecl channel-update pecl.php.net
$ pecl install apcu-4.0.11
```

Answer any question by simply pressing Return to accept the default values.

```
$ sphp 7.1
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.

```
$ sphp 7.2
$ pecl uninstall -r apcu
$ pecl install apcu
```

Answer any question by simply pressing Return to accept the default values.


# XDebug installation:

```
$ sphp 5.6
$ pecl install xdebug-2.5.5
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ open -e /usr/local/etc/php/5.6/php.ini
```

Create a new config file for XDebug:

```
$ open -e /usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
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

Restart apache:

```
$ sudo apachectl -k restart
```

In your browser go to http://localhost/info.php to ensure that XDebug is installed.


Install XDebug enable/disable script:

```
$ curl -L https://gist.githubusercontent.com/rhukster/073a2c1270ccb2c6868e7aced92001cf/raw > /usr/local/bin/xdebug
$ chmod +x /usr/local/bin/xdebug
```

Using it is simple, you can get the current state with:

```
$ xdebug
```

And then turn it on or off with:

```
$ xdebug on
$ xdebug off
```

### Xdebug for other PHP versions


### PHP 7.1

```
$ sphp 7.1
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ open -e /usr/local/etc/php/7.1/php.ini
```

Create a new config file for XDebug:

```
$ open -e /usr/local/etc/php/7.1/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
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

Restart apache:

```
$ sudo apachectl -k restart
```

### PHP 7.2

```
$ sphp 7.2
$ pecl uninstall -r xdebug
$ pecl install xdebug
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ open -e /usr/local/etc/php/7.2/php.ini
```

Create a new config file for XDebug:

```
$ open -e /usr/local/etc/php/7.2/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
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

Restart apache:

```
$ sudo apachectl -k restart
```

### PHP 7.3

```
$ sphp 7.3
$ pecl uninstall -r xdebug
$ pear config-set preferred_state beta
$ pecl install xdebug
$ pear config-set preferred_state stable
```

You will now need to remove the zend_extension="xdebug.so"" entry that PECL adds to the top of your php.ini. So edit this file and remove the top line:

```
$ open -e /usr/local/etc/php/7.3/php.ini
```

Create a new config file for XDebug:

```
$ open -e /usr/local/etc/php/7.3/conf.d/ext-xdebug.ini
```

And add the following to it:

```
[xdebug]
zend_extension="xdebug.so"
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

Restart apache:

```
$ sudo apachectl -k restart
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

To use MySQL 5.7 use the following command to start MySQL
`brew services start mysql@5.7`

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

To use MySQL 5.7 use the following command to stop MySQL
`brew services stop mysql@5.7`

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

To use MySQL 5.7 use the following command to stop MySQL
`brew services restart mysql@5.7`

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

Check the Dovecot version number. The version number `2.3.2.1` may be different but replace with the version you find.

```
$ brew info dovecot
```

The information you will get looks like this:

```
dovecot: stable 2.3.2.1 (bottled)
```

Copy the configuration files

```
$ cp -pr /usr/local/Cellar/dovecot/2.3.2.1/share/doc/dovecot/example-config/ /usr/local/etc/dovecot/
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

Check if the mail folder exists

```
ls -ltr /var/mail/YOURUSERNAME
```

If you get the message `No such file or directory` you will have to create it manually.

Create the mail folder
```
touch /var/mail/YOURUSERNAME
```

Set the permission on the mail folder. This is needed because otherwise Dovecot can't delete the messages and the log
shows the error imap(YOURUSERNAME): Error: setegid(privileged) failed: Operation not permitted. This means the 
/var/mail folder cannot be accessed by you because it is owned my root. We may need to setup LMTP as described here:
https://wiki2.dovecot.org/HowTo/PostfixDovecotLMTP

```
$ sudo chown YOURUSERNAME:mail /var/mail/YOURUSERNAME
```

Set the permissions

```
$ chmod 0600 /var/mail/YOURUSERNAME
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
User Name: YOURUSERNAME
Connection security: None
Authentication method: Password, transmitted insecurely
```

Outgoing mail

```
Server Name: localhost
Port: 25
User Name: YOURUSERNAME
Connection security: None
Authentication method: Password, transmitted insecurely
```

### Thunderbird account setup

Your name: Give the mail account a name
Email address: YOURUSERNAME@localhost
Password: Your OS X password

You can ignore the warning about double checking the email address.

Click `Continue`

Thunderbird fails to find your settings but that is OK. Fill in the details as described below

Incoming: `IMAP` `localhost` `143` `None` `Normal password`

Outgoing: `SMTP` `localhost` `25` `None` `Normal password`

Click Done

Accept the security warning that localhost does not use encryption

Click Continue